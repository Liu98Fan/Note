# SpringBoot使用Redis

之前的缓存默认是使用ConCurrentMaCacheManager,即所有的缓存数据都保存在ConCurrentMap数据结构中，实际开发中我们大多会使用很多缓存中间件，比如redis、memcached、ehcache等等；这里我们就来说一下如何使用Redis。

### 整合Redis作为缓存

#### 1.1、安装Redis

使用docker安装Redis,这里以阿里云服务器为例。

docker的安装与操纵参考前几篇笔记。

可以通过镜像加速快速拉取镜像。详情参考[docker中国](http://www.docker-cn.com/registry-mirror)

```shell
docker pull redis
```

安装成功后运行相应的镜像

```shell
 docker run -d -p 6379:6379 --name myredis docker.io/redis
```

#### 1.2、连接Redis

使用redis desktop manager进行连接，记得开放服务器的6379端口规则。

![](C:\Users\A\Desktop\笔记\我的笔记_files\2018-10-14_152226.png)

#### 1.3、Redis的一些基本操作

新建一个key-value:

```shell
append key value
#如果key存在，会在数据后面追加value
```

新建一个list，默认从队列左边入队

```shell
lpush mylist  1 2 3 4 5 6
```

从队列左边出队

```she
lpop mylist  
```

从队列右边出队

```shel
rpop mylist 
```

新建一个set,无序

```shell
sadd myset  1 2 3
```

#### 1.4、整合Redis

引入redis的starter

```xml
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

#### 1.5、配置Redis

```yam
spring:
  redis:
    host: xxxxxxxx
```

#### 1.6、使用

通过查看Redis的AutoConfigureration发现，SpringBoot自动配置了两个RedisTemplate，分别是StringRedisTemplate和RedisTemplate。

这两个Template就和JDBCTemplate的性质是一样的，是用来操作Redis的模板。

StringRedisTemplate是用来专门操作String字符串的模板，RedisTemplate则是操作对象的。

**StringRedisTemplate操作常见的五大数据类型**

- String
  - StringRedisTemplate.opsForValue()
    - StringRedisTemplate.opsForValue().append(key,value);//给Redis里保存一个key-value数据
    - StringRedisTemplate.opsForValue().get(key);//读取一个key的数据
- List
  - StringRedisTemplate.opsForList()
    - StringRedisTemplate.opsForList().leftPush(key,value[String])
- set
  - StringRedisTemplate.opsForSet()
- Hash
  - StringRedisTemplate.opsForHash()
- Zset
  - StringRedisTemplate.opsForZset()

**RedisTemplate操作对象**

- redisTemplate.opsForValue.set(key,value[object]);

但是在保存需要将对象进行序列化，所以可以让这个类实现jdk的序列化接口。(implements Serializer)

然而jdk的序列化方式保存后是二进制的乱码，所以可以有以下的解决方法:

Redis的default serializer是jdk的serializer，所以需要我们手动配置json的serializer

1、以json的方式序列化数据

例如：对于Note这个类，我们想要进行json方式序列化则需要写一个配置类：

```java
/**
 * @Author: liufan
 * @Date: 2018/10/14 15:49
 * @Description:
 */
@Configuration
public class MyRedisConfig {
    @Bean
    public RedisTemplate<Object, Note> noteRedisTemplate(
            RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
        RedisTemplate<Object, Note> template = new RedisTemplate<>();
        template.setConnectionFactory(redisConnectionFactory);
        Jackson2JsonRedisSerializer<Note> serializer = new Jackson2JsonRedisSerializer<Note>(Note.class);
        template.setDefaultSerializer(serializer);
        return template;
    }
}
```

这样以json的方式进行序列化，保存的结果就比较友好:

![](C:\Users\A\Desktop\笔记\我的笔记_files\2018-10-14_155906.png)

#### 1.7、测试缓存

Springboot缓存原理：CacheManager创建Cache缓存组件，缓存组建实际给缓存存取数据。没有开启Redis之前默认是开启SimpleCacheConfiguration，而引入Redis后，就会匹配RedisCacheConfiguration从而引入RedisCacheManager。

RedisCacheManager创建的是RedicCache缓存组件，RedisCache操作的自然是Redis。而RedisCache默认使用什么来操作Redis的呢？当然是之前提到的RedisTemplate了，不过默认的是RedisTemplate<object,object>，它的默认序列化方式还是jdkSerializer。

所以也就是说，我们需要通过重写RedisCacheManager来创建一个我们自己的使用JSON序列化方式的RedisManager。

首先我们来看以下RedisCacheConfiguration里创建默认RedisCacheManager的源码：

```java
@Bean
	public RedisCacheManager cacheManager(RedisConnectionFactory redisConnectionFactory,
			ResourceLoader resourceLoader) {
		RedisCacheManagerBuilder builder = RedisCacheManager
				.builder(redisConnectionFactory)
				.cacheDefaults(determineConfiguration(resourceLoader.getClassLoader()));
		List<String> cacheNames = this.cacheProperties.getCacheNames();
		if (!cacheNames.isEmpty()) {
			builder.initialCacheNames(new LinkedHashSet<>(cacheNames));
		}
		return this.customizerInvoker.customize(builder.build());
	}
```

**注意：这里的RedisCacheManager的创建方式是springboot2.0+版本的创建方式，和Springbot1.0x的版本有很大的不同**

```java
//此处是自定义RedisCacheManager
 @Bean
    public RedisCacheManager noteCacheManager(RedisConnectionFactory connectionFactory){
        Jackson2JsonRedisSerializer<Object> redisSerializer = new Jackson2JsonRedisSerializer<Object>(Object.class);
        RedisCacheConfiguration cacheConfiguration = RedisCacheConfiguration.defaultCacheConfig()
                .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(new
                        StringRedisSerializer()))
                .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(redisSerializer));
        RedisCacheManager redisCacheManager = RedisCacheManager.builder(connectionFactory).cacheDefaults
                (cacheConfiguration).build();
        return redisCacheManager;
    }
```

当然对于Note类的RedisCacheManager不能用于其他类，否则在读取缓存的时候会无法反序列化（这个到里应该很好理解，所以可以自定义多个CacheManager，但是一定要设置一个默认的，即在方法前加@primary）

在类前的@CacheConfig加cacheManager属性指定使用哪个manager(默认id就是manager的方法名)

所以 ，在自定义CacheManager时候模板写的是Object。

#### 1.8、不通过注解的方式操作缓存

这里有两种方式，其实我也不是很能理解，一种是通过操作RedisTemplate模板来操作缓存，还有一种是通过RedisCacheManager来操作缓存。

下面给出测试图：

![](C:\Users\A\Desktop\笔记\我的笔记_files\2018-10-15_182359.png)

可以看到两者均能操作缓存，只是当Manager获得到的myset的cache中加入一个k-v的话，通过template的get是无法获取出来的，有点蛋疼，不太理解。



