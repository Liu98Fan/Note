# GitHub笔记

### 一、版本控制：

- 集中式版本控制代表：SVN、CVS、VSS……

  ​	首先解释一下什么是集中式控制。每一个开发者是客户端，历史记录都是存储在服务器，若服务器宕机，则本地只有最近更改的数据，这就是单点故障。

  ​	SVN采用增量管理，每次将新内容与旧内容做拼接。

- 分布式版本控制代表：Git……

  ​	每一个客户端都可以在本地完成完整的版本控制，可以有效的避免单点故障。

  ​	Git采用文件快照的方式，基于文件系统的方式进行版本控制。

### 二、安装Git

- 访问Git官网
- Select Commands 保持默认设置
- 建议默认的文本编辑器：VIM
- Adjusting your path environment:选择最安全的选项，use git from git bash only
- Choosing Https transport: OpenSSL Library
- Configuring the line ending conversions:使用默认风格即可
- Configuring the terminal:使用默认控制台use MinTTY

安装完成后，右键发现git bash here

### 三、Git的本地结构

 ![](C:\Users\A\Desktop\笔记\我的笔记_files\git1.png)

#### Git和代码托管中心：

​	其任务是维护远程库

#### 那么代码托管中心一般有哪些？

- 局域网: 搭建gitlab服务器
- 外网：github、码云……

### 四、Git命令行操作

#### 1、本地库操作：

- 本地库初始化：

```shell
git init #初始化一个空的git仓库，会新建一个文件夹是.git，这是隐藏的目录
```

- 设置签名：（唯一标识提交版本的用户）

```shell
#项目级别/仓库级别：仅在当前本地库范围内有效
git config user.name XXX;
git config user.email XXX;
#系统用户级别：当前用户在系统中创建的所有仓库生效
git config --global .....
```

- 查看前面：
  - 查看.git/config文件即可

#### 2、Git的添加、提交和查看状态

- 查看状态

```shell
git status
```

- 添加到暂存区

```shell
git add
```

- 将暂存区的文件删除

```shell
git rm --cached<file>
```

- 提交文件到本地库

```shell
git commit
#提交后会让你输入信息，即本次提交的注释
#信息输入方式默认是VIM编辑器
#set nu 显示行号
#提交的时候设置注释方式如下
git commit -m "xxxxx" <file>
```

#### 3、版本控制（版本的前进和后退）

- 查看版本记录

```shell
git log #HEAD指针指向当前版本
#如果想要友好的输出
git log --pretty = oneline
git log --online
#想要全面的信息
git reflog
```

- 版本控制的三种方式

  - 基于索引值的操作

    - ```shell
      git reset --hard<索引值>#这里的索引值可以是hash值得一部分
      ```

  - 使用^符号(只能后退)

    - ```shell
      git reset --hard HEAD ^ #一个^表示后退一个版本	
      ```

  - 使用~符号（为了代替多个^，同样只可以后退）

    - ```shell
      git reset --hard HEAD~3 #后退三个版本
      ```

- reset命令的三个参数
  - --soft:仅仅在本地库移动HEAD指针
  - --mixed:在本地库移动HEAD指针并重置暂存区
  - --hard:不仅移动指针还重置工作区和暂存区

#### 4、比较文件差异

``` shell
git diff [文件名] #将工作区和暂存区的文件作比较
git diff [历史版本] [文件名] 
#不加文件名则是比较多个文件
```

### 五、分支概述

#### 1、什么是分支？

在版本控制中使用多条均线同时推进多个任务

![](C:\Users\A\Desktop\笔记\我的笔记_files\git2.png)

**可以理解为**：

​	在主分支（master）中尝试一些新的功能分别是blue和game，然后master正常上线，blue和game分别开发，互不影响，等到它们都测试完成以后合并到master主分支上线。

​	如果中途线上出了bug需要修改，则创建一个hotfix的热修分支进行bug修改，修改完以后再合并回主分支这样子的。

#### 2、分支的操作

```shell
git branch -v #查看所有分支(当前分支是绿色)
git branch XXX #添加一个XXX分支
git checkout XXX#切换到XXX分支
#当分支有修改以后
#需要合并分支，执行以下步骤：
#①切换到接受修改的分支上
#②执行
git merge[需要合并的分支名]
```

#### 3、解决冲突

当两个分支同一个文件中的同一个位置被修改时会产生冲突

1. 编辑文件，删除特殊符号：![](C:\Users\A\Desktop\笔记\我的笔记_files\git3.png)

2. 修改文件

3. ```shell
   git add [文件名]
   ```

4. ```shell
   git commit -m "注释" #注意这时候一定不要加文件名
   ```

### 六、github的注册和创建远程库

创建啥的我就不记了，这些按照网站引导都OKder~

#### 1、创建远程库

```shell
git remote -v #查看地址别名
git remote add [别名] [地址] #给地址起别名 ，不然地址太长了很麻烦
git push [别名/地址] [分支] #推送分支到远程库
```

#### 2、克隆操作

```shell
git clone [远程库地址]
```

