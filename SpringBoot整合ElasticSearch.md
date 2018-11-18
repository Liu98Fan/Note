# SpringBoot整合ElasticSearch

### 一、安装ElasticSearch

首先要确定版本匹配，不然会出很多问题，因为springBoot推荐使用springdata整合elasticsearch，所以进入springdata的官网查找到当前版本<https://github.com/spring-projects/spring-data-elasticsearch>

然后就是依旧用docker 安装

```shell
docker pull registry.docker-cn.com/library/elasticsearch
```

安装成功后运行镜像

```shell
docker run -d -p 9200:9200 -p 9300:9300 -e ES_JAVA_OPTS="-Xms256m -Xmx256m" --name XXX {imageID}
#-e参数设置JVM分配的内存，由于elasticSearch默认分配2G内存，很多服务器内存不足，启动失败
```

启动完成后使用命令查看是否成功启动：

```shel
docker ps -a
```

如果启动失败，通过日志查看错误信息

```shell
docker logs {containerId}
```

最后不要忘了服务器安全组打开相应端口权限

### 二、使用ElasticSearch

坑先不填，以后再说



