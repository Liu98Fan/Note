# Mysql存储过程加事务操作

首先场景如下：

一张笔记表(note_tb)

![](C:\Users\A\Desktop\笔记\我的笔记_files\2018-10-15_193815.png)

一张笔记分类表(noteclassify_tb)

![](C:\Users\A\Desktop\笔记\我的笔记_files\2018-10-15_193906.png)

**网站中插入笔记需要查询插入的笔记在表中存不存在，即分辨是更新操作还是插入操作，同时区分笔记分类是否存在，如果不存在则新建此分类。**

课程学的是sql，对于mysql的事务管理和存储过程在语言上似乎还存在着一些障碍，有些语法不太相同，首先说一个坑：

在navicat中尝试运行此代码，总是报语法错误：

![](C:\Users\A\Desktop\笔记\我的笔记_files\2018-10-15_194124.png)

真的是非常的不解！

尝试很多办法，最后使用mysql workbench查看细致的错误发现是分隔符的原因，mysql默认是分号(;)作为语句结束符，而写代码习惯了写完一条语句就是分号，会在mysql中导致语法错误，解决办法就是修改以下默认的结束符。

```mysql
delimiter |
```

然后就是存储过程和事务处理的声明和删除啥的，比较基础就不多赘述了。主要记录一下在事务处理中进行回滚的判断，可以通过一个变量，当出现exception时候变动它的值，从而通过判断这个变量的值来决定是提交还是回滚，下面给出具体的代码：

```mysql
delimiter |
CREATE PROCEDURE saveNote (IN p_id VARCHAR(32),IN p_title VARCHAR(255),IN p_author VARCHAR(255) ,IN p_content text,IN p_date varchar(255),IN p_newdate varchar(255),IN p_del_flag INT,IN p_classifyid varchar(32),IN p_classifyName VARCHAR(255)  )
BEGIN
	DECLARE var int ;
	DECLARE classify_var INT;
	DECLARE error_code INT;
    DECLARE CONTINUE HANDLER FOR SQLEXCEPTION SET error_code=1; 
	SET var = (select count(*) from note_tb where id = p_id and del_flag = 0);
	SET classify_var = (select count(*) from noteclassify_tb WHERE id = p_classifyid and del_flag = 0);
	
	
	-- 开始事物
	START TRANSACTION;
		select classify_var ;
		if classify_var < 1 THEN
			-- 如果classify_tb中没有分类数据,则执行插入，表示这个分类是不存在的，如果存在则不用管
			INSERT INTO noteclassify_tb VALUES(p_classifyid,p_classifyName,p_del_flag,p_date);
            select "insert class";
		END IF;
		select var ; 
		if var > 0 THEN
			-- 如果note表中有数据，则执行更新
			update note_tb
			SET	
					title = p_title,content = p_content,author = p_author,newdate = p_newdate,del_flag = p_del_flag,classifyid = p_classifyid
			where id = p_id and del_flag = 0;
			select "update note";
		ELSE
			insert into note_tb values(p_id,p_title,p_author,p_content,p_classifyid,p_date,p_newdate,p_del_flag);
            select "insert note";
		end IF;

		-- 提交事务
		
		IF error_code = 1 THEN
				ROLLBACK;
				select "error";
		ELSE
				COMMIT;
                select "succ";
		END IF;
	
	select * from note_tb where id = p_id and del_flag = 0;
END|
```

以上可以正确的运行。