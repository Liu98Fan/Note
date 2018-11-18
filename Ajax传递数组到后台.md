### Ajax传递数组到后台

今天遇到了一个问题，在SSM框架下，前端使用jquery的$.ajax({})传递string 数组到后台，发现无法接受数据，一直是null。折磨了我两天，在朋友的帮助下，打了N多个断点，发现，当数据从前端通过get方式传递到后台时候，数组数据总是会多了一点东西。

比如 一个titleList=['1','1'];通过data:{title:titleList}传递后，总会出现title%5B%5D=****的情况，随后从网上查找资料发现，jquery的ajax在解析数据的时候会默认深度解析，即traditional的属性默认是false,

当传递数组数据的时候必须traditional:true,否则数组数据会被过度解析从而后台无法接受正确的数据。

官方文档的解释如下：

```java
traditional 

类型：Boolean

如果你想要用传统的方式来序列化数据，那么就设置为 true。

Set this to true if you wish to use the traditional style of param serialization
```

### windows下的Mysql中文乱码问题

今天在做简单的数据库插入的时候发现如下问题：

1、网站提交的数据到后台断点发现中文显示正常

2、navicat里写查询语句insert插入中文正常

3、网站提交的数据插入数据库发现中文都是?，乱码了



碰到此类问题首先cmd 进入控制台 mysql -u root -p进入mysql

输入:show variables like '%char%';  查看数据库的编码

```java
show variables like '%char%';

+--------------------------+---------------------------------------------------------+

| Variable_name            | Value                                                   |

+--------------------------+---------------------------------------------------------+

| character_set_client     | gbk                                                     |

| character_set_connection | gbk                                                     |

| character_set_database   | latin1                                                  |

| character_set_filesystem | binary                                                  |

| character_set_results    | gbk                                                     |

| character_set_server     | latin1                                                  |

| character_set_system     | utf8                                                    |

| character_sets_dir       | C:\Program Files\MySQL\MySQL Server 5.7\share\charsets\ |

+--------------------------+---------------------------------------------------------+
```

发现存在latin的编码，应当改为utf8

网上的set -character=utf8是无效的，为啥呢，因为mysql启动时候的编码是根据配置文件来的，set只是修改当前进程mysql的编码，无法做到一劳永逸，所以修改编码应当修改配置文件。

找到my.ini（找不到可以百度），添加如下（或者修改如下）

```properties
[mysqld] 
character-set-server=utf8 
[client] 
default-character-set=utf8  
[mysql] 
default-character-set=utf8
```

然后重启mysql服务即可