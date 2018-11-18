# 腾讯云从0部署springboot项目

1、首先你得买一个服务器，我装得是ubuntu16.04得系统，然后腾讯云会帮你生成一个服务器实例。

这个服务器得默认登陆用户名是ubuntu，密码在站内信中，是一串随机的字符串，可以自己修改。

2、想要获取服务器的root权限需要设置root的初始密码,进行如下操作设置密码:

```shell
sudo passwd root
```

3、然后就可以通过su root 来切换到管理员权限

但是，这里还有一个问题，SSH连接的时候无法通过root来登陆，需要自己配置以下配置文件:

```she
vi /etc/ssh/sshd_config
```

找到PermitRootLogin,将后面的***-password替换成yes即可

然后保存退出，重启ssh服务

```she
service ssh restart
```

4、腾讯云默认是没有JDK的，所以:

```sh
sudo apt-get install openjdk-8-jdk
```

5、打包好jar,在服务器上进行如下操作:

```sh
nohup java -jar springboot-mywebsite-0.0.1-SNAPSHOT.jar --server.port=80 &
# nohup是指定操作后台运行  
# &表示信息默认输出
```

6、查看端口运行情况:

```s
lsof -i:80(端口号)

#关闭进程
skill PID
```





