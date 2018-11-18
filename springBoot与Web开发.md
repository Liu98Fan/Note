# springBoot与Web开发

## 1、SpringBoot对静态资源的映射规则

### 1.1、对公共资源的映射规则

```java
public void addResourceHandlers(ResourceHandlerRegistry registry) {
    if (!this.resourceProperties.isAddMappings()) {
        logger.debug("Default resource handling disabled");
    } else {
        Duration cachePeriod = this.resourceProperties.getCache().getPeriod();
        CacheControl cacheControl = this.resourceProperties.getCache().getCachecontrol().toHttpCacheControl();
        if (!registry.hasMappingForPattern("/webjars/**")) {
            this.customizeResourceHandlerRegistration(registry.addResourceHandler(new String[]{"/webjars/**"}).addResourceLocations(new String[]{"classpath:/META-INF/resources/webjars/"}).setCachePeriod(this.getSeconds(cachePeriod)).setCacheControl(cacheControl));
        }
```

​	可以看出静态资源的一种映射规则是：registry.hasMappingForPattern("/webjars/**")，即"/webjars/\*\*"

，代码中指出：所有的/webjars/**，都去classpath:/META-INF/resources/webjars/找资源

​	那么webjars是什么？

​	webjars是以jar包的方式引入静态资源；可以通过maven导入的方式导入像jquery啊bootstrap等这些静态框架资源，具体可以参考webjars的官方文档：https://www.webjars.org/

​	在pom.xml中加入如下声明：

```xml
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>jquery</artifactId>
    <version>3.3.1-1</version>
</dependency>
```

​	查看导入的jar包发现如下图:

![](C:\Users\A\Desktop\笔记\我的笔记_files\webjars-jquery.png)



jquery的资源就被成功引入到项目中去了。

​	比如我发一个请求：localhost:8080/webjars/abc，SpringBoot会去classpath:/META-INF/resources/webjars/abc寻找资源。

### 1.2、对个人资源的映射规则

​	公共的框架可以通过webjars引入，那么自己的一些CSS或者图片如何引入呢？我们可以通过SpringBoot的代码来看一下是如何定义的：

```java

```

```java
public WebMvcProperties() {
        this.localeResolver = WebMvcProperties.LocaleResolver.ACCEPT_HEADER;
        this.dispatchTraceRequest = false;
        this.dispatchOptionsRequest = true;
        this.ignoreDefaultModelOnRedirect = true;
        this.throwExceptionIfNoHandlerFound = false;
        this.logResolvedException = false;
        this.staticPathPattern = "/**";
```

发现另的一种staticPathPattern默认是"/\*\*"	

任何路径访问当前项目的任何资源都是去以下几个地方寻找：

```java
private static final String[] CLASSPATH_RESOURCE_LOCATIONS = new String[]{
	"classpath:/META-INF/resources/", 
    "classpath:/resources/",                                                               "classpath:/static/",                                                                   "classpath:/public/"
};
```

加上/**一共五种路径：

+ "classpath:/META-INF/resources/"
+ "classpath:/resource"
+ "classpath:/static/"
+ "classpath:/public/"
+ "/**"

这些目录有些是有的，有些则需要自己创建，如下：![](C:\Users\A\Desktop\笔记\我的笔记_files\静态资源文件夹.png)

​	那么假设我们要访问某一个资源的时候可以通过localhost:8080/abc 去以上五个资源文件夹里面找abc

### 1.3、欢迎页的映射规则

```java
@Bean
public WelcomePageHandlerMapping welcomePageHandlerMapping(ApplicationContext applicationContext) {
    return new WelcomePageHandlerMapping(new TemplateAvailabilityProviders(applicationContext), applicationContext, this.getWelcomePage(), this.mvcProperties.getStaticPathPattern());
}
//getWelcomePage()得到欢迎页
//getStaticPathPattern()得到映射规则
```

​	深入getWelcomePage()发现：

```java
private Optional<Resource> getWelcomePage() {
    String[] locations = getResourceLocations(this.resourceProperties.getStaticLocations());
    return Arrays.stream(locations).map(this::getIndexHtml).filter(this::isReadable).findFirst();
}

private Resource getIndexHtml(String location) {
    return this.resourceLoader.getResource(location + "index.html");
}
```

​	从locations里获取"index.html"作为欢迎页，getStaticLocations()的结果也就是上面提到的那四种默认的静态资源文件夹。

​	也就是说，当我们访问localhost:8080/时，应当跳转欢迎页，欢迎页则去静态资源默认的文件夹下寻找index.html作为欢迎页。

### 1.4、图标映射规则

​	这里就不贴代码了，按照上述分析方法，发现，所有的图标资源"**/favicon.ico"也是在静态资源文件夹下寻找。

### 1.5、修改静态资源文件夹的路径

```java
/**
由于staticLocations定义在ResourceProperties配置类中，
发现 prefix = "spring.resources",
想要修改staticLocation，则可以通过配置文件配置spring.resources.static-location的值即可
**/
@ConfigurationProperties(
    prefix = "spring.resources",
    ignoreUnknownFields = false
)
public class ResourceProperties {
    private static final String[] CLASSPATH_RESOURCE_LOCATIONS = new String[]{"classpath:/META-INF/resources/", "classpath:/resources/", "classpath:/static/", "classpath:/public/"};
    private String[] staticLocations;
    private boolean addMappings;
    private final ResourceProperties.Chain chain;
    private final ResourceProperties.Cache cache;

    public ResourceProperties() {
        this.staticLocations = CLASSPATH_RESOURCE_LOCATIONS;
        this.addMappings = true;
        this.chain = new ResourceProperties.Chain();
        this.cache = new ResourceProperties.Cache();
    }
```

​	可以作如下配置:

```properties
spring.resources.static-locations=classpath:/hello/,classpath:/liufan/
#由于这个量是个数组，所以可以配置多个路径，用逗号隔开即可。
```



## 2、模板引擎

### 2.1、什么是模板引擎

springBoot默认是不支持jsp的，如果不使用jsp，只使用html，会非常不方便，所以SpringBoot建议使用模板引擎。实JSP也是一种模板引擎，只不过由于各种原因，springboot默认不支持了。

​	那么什么叫模板引擎？其

​	下面来先看一张图：

​	![](C:\Users\A\Desktop\笔记\我的笔记_files\template-engine.png)

​	所谓模板引擎，参考上述图片解释就是，首先定义一个Template，其中有一些变量是动态的，需要后台传入，访问的时候通过后台得到数据后，通过模板引擎将动态数据和模板结合组装成output，即我们看到的页面，这就是模板引擎实现的功能。

​	springBoot官方推荐的模板引擎是:thymeleaf.

### 2.2、thymeleaf

#### 2.2.1、引入thymeleaf

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

#### 2.2.2、thymeleaf使用示例

​	首先我们来看一下springBoot对于thymeleaf的自动配置:

```java
@ConfigurationProperties(
    prefix = "spring.thymeleaf"
)
public class ThymeleafProperties {
    private static final Charset DEFAULT_ENCODING;
    public static final String DEFAULT_PREFIX = "classpath:/templates/";
    public static final String DEFAULT_SUFFIX = ".html";
    private boolean checkTemplate = true;
    private boolean checkTemplateLocation = true;
    private String prefix = "classpath:/templates/";
    private String suffix = ".html";
    private String mode = "HTML";
```

​	只要我们把HTML页面放在DEFAULT_PREFIX = "classpath:/templates/"下即可访问相应的页面。

​	例如，我在templates文件夹下创建一个test.html页面，现在我们来写一个例子：

```java
/**
 * 原来SpringMVC会将返回的success进行前后缀封装后定位到相应的页面
 * 现在SpringBoot也一样，加上前缀classpath:/templates/,后缀.html
 * 会定位到classpath:/templates/test.html页面
 * @return
 */
@RequestMapping("/test")
public String test(){
    return "test";
}
```

​	这个页面就是thymeleaf来自动渲染的。

#### 2.2.3、引入thymeleaf的命名空间

```html
<html xmlns:th="http://www.thymeleaf.org">
```

这样可以拥有thymeleaf的自动提示功能

thymeleaf的语法详情可以查看[thymeleaf的官方文档](./usingthymeleaf.pdf)

#### 2.2.4、常用语法

+ ##### th:text   

```java
<p th:text="#{home.welcome}">Welcome to our grocery store!</p>
```

​	#{}：从配置文件中获取数据

​	${} ：从域中获取数据

+ ##### th:utext

```java
<p th:utext="#{home.welcome}">Welcome to our grocery store!</p>
```

​	解析html标签，不会对其进行转义

+ 其他：​	![](C:\Users\A\Desktop\笔记\我的笔记_files\2018-02-04_123955.png)

```properties
1、Simple expressions:（表达式语法）
    Variable Expressions: ${...}：获取变量值；OGNL；
    		1）、获取对象的属性、调用方法
    		2）、使用内置的基本对象：
    			#ctx : the context object.
    			#vars: the context variables.
                #locale : the context locale.
                #request : (only in Web Contexts) the HttpServletRequest object.
                #response : (only in Web Contexts) the HttpServletResponse object.
                #session : (only in Web Contexts) the HttpSession object.
                #servletContext : (only in Web Contexts) the ServletContext object.
                
                ${session.foo}
            3）、内置的一些工具对象：
#execInfo : information about the template being processed.
#messages : methods for obtaining externalized messages inside variables expressions, in the same way as they would be obtained using #{…} syntax.
#uris : methods for escaping parts of URLs/URIs
#conversions : methods for executing the configured conversion service (if any).
#dates : methods for java.util.Date objects: formatting, component extraction, etc.
#calendars : analogous to #dates , but for java.util.Calendar objects.
#numbers : methods for formatting numeric objects.
#strings : methods for String objects: contains, startsWith, prepending/appending, etc.
#objects : methods for objects in general.
#bools : methods for boolean evaluation.
#arrays : methods for arrays.
#lists : methods for lists.
#sets : methods for sets.
#maps : methods for maps.
#aggregates : methods for creating aggregates on arrays or collections.
#ids : methods for dealing with id attributes that might be repeated (for example, as a result of an iteration).

2、Selection Variable Expressions: *{...}：选择表达式：和${}在功能上是一样；
    	补充使用：配合 th:object="${session.user}：
         例子:   
         		<div th:object="${session.user}">
                    <p>Name: <span th:text="*{firstName}">Sebastian</span>.</p>
                    <p>Surname: <span th:text="*{lastName}">Pepper</span>.</p>
                    <p>Nationality: <span th:text="*{nationality}">Saturn</span>.</p>
                </div>
    
3、Message Expressions: #{...}：获取国际化内容
4、Link URL Expressions: @{...}：定义URL；
    		@{/order/process(execId=${execId},execType='FAST')}
5、Fragment Expressions: ~{...}：片段引用表达式
    		<div th:insert="~{commons :: main}">
    		
    		</div>
            
Literals（字面量）
      Text literals: 'one text' , 'Another one!' ,…
      Number literals: 0 , 34 , 3.0 , 12.3 ,…
      Boolean literals: true , false
      Null literal: null
      Literal tokens: one , sometext , main ,…
Text operations:（文本操作）
    String concatenation: +
    Literal substitutions: |The name is ${name}|
Arithmetic operations:（数学运算）
    Binary operators: + , - , * , / , %
    Minus sign (unary operator): -
Boolean operations:（布尔运算）
    Binary operators: and , or
    Boolean negation (unary operator): ! , not
Comparisons and equality:（比较运算）
    Comparators: > , < , >= , <= ( gt , lt , ge , le )
    Equality operators: == , != ( eq , ne )
Conditional operators:条件运算（三元运算符）
    If-then: (if) ? (then)
    If-then-else: (if) ? (then) : (else)
    Default: (value) ?: (defaultvalue)
Special tokens:
    No-Operation: _ 
```





























