# SpringBoot缓存

### 1、引入

```xml
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
```

### 2、开启缓存注解模式

在运行类上写上一个注解@EnableCaching

```java
@SpringBootApplication
@EnableCaching
public class SpringbootMywebsiteApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootMywebsiteApplication.class, args);
    }
}
```

### 3、缓存的使用

#### 3.1、缓存模型

![](C:\Users\A\Desktop\笔记\我的笔记_files\cach.png)

首先解释一下这张图:

​	Entry<K,V>:存储在Cache中的key-value对

​	Expiry:每一个key-value都有一个有效期，一旦过期该条目则不可访问、更新和删除

然后很多个Entry组合成为一个catch,怎么理解呢？其实就是一些有相同特征的Entry的聚合，实际使用过程总是将一类的缓存Entry放在一个Cach中。

​	Cache:是一个类似Map的数据结构并临时存储以Key为索引的值。

然后很多个Cache被一个CacheMannager管理，然后很多个CacheMannager被一个CachingProvider拥有。

#### 3.2、缓存使用

##### @Cacheable注解

举个栗子，前台页面每次访问都会有Ajax向后台发送请求描述信息，每次刷新页面都会执行getDescription方法，那么可以在方法前加上一个@Cacheable的注解，指定好cacheNames即cache的名字。默认会将方法的返回值进行缓存。默认的key是方法参数的值。

```java
    @RequestMapping(value = "/getDescription")
    @ResponseBody
    @Cacheable(cacheNames = {"description"})
    public Object getDescription(@RequestParam(value = "mark") String mark){
        if (mark!=null){
             JSONArray jsonArray = centerService.getDescription(Integer.parseInt(mark));
            if (jsonArray!=null){
                return jsonArray;
            }else{
                return "error";
            }
        }else {
            return "error";
        }
    }
```

其他属性:

- cacheNames/Value:指定缓存组件的名字，将数据保存在那个缓存中，是数组的方式，可以指定多个缓存

- key:缓存数据使用的key,下次查询缓存的时候通过key来查询需要的值.默认是方法参数的值。支持spel表达式.

  - 若存在多个参数，想要使用其中一个参数座位key，可以通过#id来获取参数名为id的值作为key,#a0、#p0、#root.args[0]来获取第一个参数的值作为key。
  - Spel表达式中可以通过一些对象来获取方法的一些数据。
    - 例如:对于getDescription（String mark）方法，想要getDescription[1]作为key,可以使用key="#root.getMethodName+'['+#mark+']'"来拼接一个key.

- keyGenerator:和key属性二选一使用，用来指定key的生成器组件id（生成器组件可以自定义）

  - 还可以通过自定义一个keyGenerator来写一个key生成器。

    - ```java
      @Configuration
      public class MyCacheConfig{
          @Bean('myKeyGenerator')
          public KeyGenerator keyGenerator(){
              return new KeyGenerator(){
                  @Override
                  public Object generate(Object target,Method method,Object... 													params){
              		return method.getName()+"					["+Arrays.asList(params).toString+"]";        
                  }
              }
          }
      }
      ```

- cacheManager：指定使用哪个缓存管理器

- condition:指定条件下才进行缓存

  - 例如：condition="#id>0" 当参数里的id大于0时候对结果进行缓存

- unless:否定缓存，指定条件就不进行缓存,和condition正好相反
- sync:是否使用异步模式，异步模式下，unless不支持。

##### @cacheable原理

1. 方法运行前，先去查询Cache缓存组建，按照CacheNames指定的名字获取。如果是第一次获取缓存会自动创建这个名字的cache

2. 去Cache中查找缓存的内容，根据key。默认就是方法的参数。key是按照某种策略生成。默认是用keyGenarator生成，keyGenerator默认使用SimpleKeyGenerator生成key;

   ​	SimpleKeyGenerator生成key的默认策略：

   ​			如果没有参数：key=new SimpleKey()；

   ​			如果有一个参数：key = 参数的值

   ​			如果有多个参数: key = new SimpleKey(params)

3. 如果没有查到缓存就调用目标方法；

4. 将目标方法返回的结果，放入缓存中。

##### @CachePut注解

既调用方法，又更新缓存。一般用在更新数据库的方法上.使用方法同@Cacheable

运行时机在方法调用后，更新缓存数据。因此，key指定的时候可以通过#result来指定方法结果的值作为key。而@Cacheable不可以使用，因为@Cacheable在方法调用之前就需要运行。

**？思考:缓存的都是方法返回值，如何指定缓存的数据？因为很多时候需要缓存的数据并不是方法的返回值。朋友说注解的属性好像没有此类功能。需要手动写入缓存。**

##### @CacheEvict注解

缓存清除,一般在方法是删除数据库数据的时候使用。运行时机在方法调用后，使用方法同@Cacheable。也有一些不同的属性：

- allEntries:为true时，会将指定的cache的所有数据都清除。默认为false.
- beforeInvocation:是否在方法之前执行，默认时false

##### @Caching注解

这是Cacheable CachePut 和CacheEvict的组合注解。

```java
@Caching(

	cacheable = {	

		@Cacheable(value="emp"，key = "#lastName")

	}

	put = {	

		@CachePut(value="emp",key = "#result.id")

		@CachePut(value="emp",key = "#result.email")

	}

)
public object XXXX(){}
//就是说可以让一个方法拥有多个缓存注解，来定义复杂的缓存规则。
```

##### @CacheConfig注解

在类上写这个注解

指定CacheNames后，类中所有的方法就默认是同一个的CacheNames了。



