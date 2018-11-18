# docker安装mysql

安装docker (centos 7)

```shell
1、安装docker
yum install docker
2、启动docker
systemctl start docker
3、查看docker镜像
docker images
4、删除docker镜像
docker rmi #{image_ID}
```

安装mysql

​	可以去[docker hub](https://hub.docker.com/) 的官网查看MySQL的官方文档

```shell
1、搜索镜像
docker search mysql
2、下载镜像，如果默认latest会由于mysql8.0版本得密码新特性导致连接报错，所这里下载5.7
docker pull mysql：5.7
3、运行镜像,根据官方文档
docker run -p 3306:3306 --name mysql01 -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7
4、配置编码得方式运行镜像
docker run --name my_mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=60085040 -d mysql:5.7 --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci

```

docker 的其他操作

```shell
1、查看当前运行的容器
docker ps
2、查看所有容器（运行和非运行的）
docker ps -a 
3、删除容器
docker rm #{container_ID}
4、查看容器日志（比如对于mysql的容器就可以看到mysql的日志）
docker logs #{container_ID}
5、 -v参数可以用来映射文件夹
	-v /etc/my01:/my02
	代表将本地的/etc/my01文件夹映射到容器的my02文件夹

```

