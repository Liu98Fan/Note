## 基于Maven搭建SSM+Shiro框架

### 一、环境选择

**开发工具：**Eclipse

**系统环境：**Windows 10

**框架选型：**Spring+SpringMVC+Mybatis+Shiro

### 二、搭建过程

#### 1、创建Maven工程

![](C:\Users\A\Desktop\笔记\我的笔记_files\SSM+shiro1.png)

![](C:\Users\A\Desktop\笔记\我的笔记_files\SSM+shiro2.png)

![](C:\Users\A\Desktop\笔记\我的笔记_files\SSM+shiro3.png)

![](C:\Users\A\Desktop\笔记\我的笔记_files\SSM+shiro4.png)

**至此，我们的一个空的maven工程就创建完毕！**

#### 2、引入依赖

**首先我们来看看eclipse中的maven如何快速引入依赖：**

![](C:\Users\A\Desktop\笔记\我的笔记_files\SSM+shiro5.png)

**其次我们再来看看这个框架需要哪些依赖以及他们的作用**

| 名称                       | 解释                                                         |
| :------------------------- | :----------------------------------------------------------- |
| **spring-aop **            | 这个jar 文件包含在应用中使用Spring 的AOP 特性时所需的类和源码级元数据支持。使用基于AOP 的Spring特性，如声明型事务管理（Declarative Transaction Management），也要在应用里包含这个jar包。**外部依赖spring-core**。 |
| **spring-aspects**         | 提供对AspectJ的支持                                          |
| **spring-context**         | 这个jar 文件为Spring 核心提供了大量扩展。可以找到使用Spring ApplicationContext特性时所需的全部类，JDNI 所需的全部类，instrumentation组件以及校验Validation 方面的相关类。**外部依赖spring-beans** |
| **spring-context-support** | 包含支持缓存Cache（ehcache）、JCA、JMX、 邮件服务（Java Mail、COS Mail）、任务计划Scheduling（Timer、Quartz）方面的类。**外部依赖spring-context** |
| **spring-core**            | 这个jar 文件包含Spring 框架基本的核心工具类。Spring 其它组件要都要使用到这个包里的类，是其它组件的基本核心，当然你也可以在自己的应用系统中使用这些工具类。**外部依赖Commons Logging， (Log4J)。** |
| **spring-test**            | 对Junit等测试框架的简单封装。                                |
| **spring-web**             | 这个jar 文件包含Web 应用开发时，用到Spring 框架时所需的核心类，包括自动载入Web ApplicationContext 特性的类、Struts 与JSF 集成类、文件上传的支持类、Filter 类和大量工具辅助类。 |
| **spring-webmvc**          | 这个jar 文件包含Spring MVC 框架相关的所有类。包括框架的Servlets，Web MVC框架，控制器和视图支持。当然，如果你的应用使用了独立的MVC 框架，则无需这个JAR 文件里的任何类。**外部依赖spring-web** |
| **mybatis**                | mybatis核心包                                                |
| **mybatis-spring**         | mybatis和spring整合包                                        |
| **log4j**                  | 日志                                                         |
| **slf4j-api**              | 日志                                                         |
| **slf4j-log4j**            | 日志                                                         |
| **mysql-connector-java**   | mysql驱动包                                                  |
| **jstl**                   | jstl标签                                                     |
| **servlet-api**            | servlet支持                                                  |
| **java.servlet-api**       | servlet支持                                                  |
| **commons-dbcp**           | 主流数据库连接池                                             |

#### 3、编写配置文件

##### **web.xml**

- 编码过滤器

  - 很简单很实用的一个过滤器，当前台JSP页面和JAVA代码中使用了不同的字符集进行编码的时候就会出现表单提交的数据或者上传/[下载](http://www.2cto.com/soft)中文名称文件出现乱码的问题，这就是解决这个问题的。

    ```xml
    <!-- 首先编码过滤器 -->
    <filter>
    	<filter-name>encodingFilter</filter-name>
    	<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    	<async-supported>true</async-supported>
    	<init-param>
    		<param-name>encoding</param-name>
    		<param-value>UTF-8</param-value>
    	</init-param>
    </filter>
    <!-- 然后配置过滤去拦截地址 这里选则/*表示将所有资源数据编码都进行过滤-->
    <filter-mapping>
    	<filter-name>encodingFilter</filter-name>
    	<url-pattern>/*</url-pattern>
    </filter-mapping>
    
    ```


- spring监听器

  - 用来在web容器启动时候自动加载spring的配置文件中的信息,其默认会扫描WEB-INF下的applicationContext.xml配置文件，所以一般spring的默认配置文件都是这个名字，如果将其改名并放到classpath下的话就需要进行一个配置了。奇怪的是，这个变量的配置并不需要在固定的位置，我在SpringMVC的servlet中配置它的路径也生效了。

    ```xml
    <!-- 首先加载listener -->
    <listener>
    	<listener-class>
                org.springframework.web.context.ContextLoaderListener
        </listener-class>
    </listener>
    ```

- SpringMVC的servlet

  - 该配置用来配置springMVC的路径解析器，即DispatcherServlet。配置后，相应请求将会被springMVC拦截到。

    ```xml
    <!-- Spring MVC servlet -->
    	<servlet>
    		<servlet-name>SpringMVC</servlet-name>
    		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    		<init-param>
    			<param-name>contextConfigLocation</param-name>
    			<param-value>classpath:spring-mvc.xml</param-value>
    		</init-param>
    		<load-on-startup>1</load-on-startup>
    		<async-supported>true</async-supported>
    	</servlet>
    <!-- 当然同样的要配置servlet的mapping地址来说明它拦截哪些路径 -->
    	<servlet-mapping>
    		<servlet-name>SpringMVC</servlet-name>
    		<!-- 放弃了.do和.action的风格 -->
    		<url-pattern>/</url-pattern>
    	</servlet-mapping>
    ```


- Shiro的过滤器配置

  - 实际上这是Filter的一个代理，默认情况下，spring会到IOC容器中查找与\<filter-name\>一致的Bean，也可以通过\<init-param\>的<param-name\>来指定Bean。

    ```xml
    <filter>  
        <filter-name>shiroFilter</filter-name>  
        <filter-class>
            org.springframework.web.filter.DelegatingFilterProxy
        </filter-class>  
        <init-param>  
             <!-- 该值缺省为false,表示生命周期由SpringApplicationContext管理,设置为true则表示由ServletContainer管理 -->  
            <param-name>targetFilterLifecycle</param-name>  
            <param-value>true</param-value>  
        </init-param>  
    </filter> 
    
    <filter-mapping>  
        <filter-name>shiroFilter</filter-name>  
        <url-pattern>/*</url-pattern>  
    </filter-mapping>
    ```

- SpringMvc配置静态资源不拦截

  - 默认情况下SpringMVC按照/将会拦截所有请求，包括静态资源的请求。这是不对的，所以通过springMVC中的配置mvc:default-servlet-handler来配置一个default的servlet的mapping来指定一些静态资源后缀的请求可以不被拦截。

    ```xml
    <!-- 过滤静态资源，从而不拦截 -->
    	<servlet-mapping>
    		<servlet-name>default</servlet-name>
    		<url-pattern>*.js</url-pattern>
    		<url-pattern>*.css</url-pattern>
    		<url-pattern>/imgs/*</url-pattern>
    		<url-pattern>*.jpg</url-pattern>
    		<url-pattern>*.woff</url-pattern>
    		<url-pattern>*.tff</url-pattern>
    	</servlet-mapping>
    ```

- spring配置文件读取

  ```xml
  <!-- spring配置文件加载 -->
  <context-param>
  	<param-name>contextConfigLocation</param-name>
  	<param-value>classpath:spring-*.xml</param-value>
  </context-param>
  ```

##### spring-mvc.xml

​	配置完web.xml中内容以后既可以来配置spring了，在项目路径下创建一个spring-mvc.xml的文件，这里名字怎么取都可以，因为我们在web.xml里面要配置contextConfigLocation的值，其对应了spring的配置文件路径，so，随便取吧，这里我根据我的web.xml里面配的值创建文件。

- 首先引入命名空间

  ```xml-dtd
  xmlns="http://www.springframework.org/schema/beans"  
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"  
      xmlns:context="http://www.springframework.org/schema/context"  
      xmlns:mvc="http://www.springframework.org/schema/mvc"  
      xsi:schemaLocation="http://www.springframework.org/schema/beans    
                          http://www.springframework.org/schema/beans/spring-beans-3.1.xsd    
                          http://www.springframework.org/schema/context    
                          http://www.springframework.org/schema/context/spring-context-3.1.xsd    
                          http://www.springframework.org/schema/mvc    
                          http://www.springframework.org/schema/mvc/spring-mvc-4.0.xsd"
  ```

- 静态资源处理器

  - 还记得web.xml配置的静态资源不拦截的servlet么，没错，那里配置的就是这里的servlet的参数。

    ```xml
        <!-- 静态资源处理 -->
        <mvc:default-servlet-handler/>
    ```


- 开启各种注解方式

  ```xml
       <!-- 配置注解的映射器和适配器 -->
   <mvc:annotation-driven ></mvc:annotation-driven>
  ```

- 开启注解以后得发现各种注解呀对吧。

  ```xml
  <context:component-scan base-package="父包名" />  
  ```

- 开启SpringMVC的注解功能和映射功能

  ```xml
      <!-- 启动SpringMVC的注解功能，完成请求和注解POJO的映射 -->  
  <bean   class="org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter">  
      </bean>  
  ```

- 配置视图解析器解析模板文件地址

  ```xml
  <!-- 定义跳转的文件的前后缀 ，视图模式配置-->  
  <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">  
      <!-- 这里的配置我的理解是自动给后面action的方法return的字符串加上前缀和后缀，变成一个 可用的url地址 -->  
      <property name="prefix" value="/" />  
      <property name="suffix" value=".jsp" />  
  </bean> 
  ```

##### spring-mybatis.xml

​	接下来就是配置数据持久层框架的配置，其实就是一些数据源等等的配置

- 命名空间

  ```xml-dtd
  xmlns="http://www.springframework.org/schema/beans"
  	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
  	xmlns:context="http://www.springframework.org/schema/context"
  	xmlns:mvc="http://www.springframework.org/schema/mvc"
  	xsi:schemaLocation="http://www.springframework.org/schema/beans    
                          http://www.springframework.org/schema/beans/spring-beans.xsd    
                          http://www.springframework.org/schema/context    
                          http://www.springframework.org/schema/context/spring-context.xsd    
                          http://www.springframework.org/schema/mvc    
                          http://www.springframework.org/schema/mvc/spring-mvc.xsd"
  ```

- 配置数据源

  - 这里我们推荐将数据源信息写在一个properties文件中，直接加载进xml

    ```xml
    <!--加载信息文件-->	
    <bean id="propertyConfigurer"
    		class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
    		<property name="location" value="classpath:jdbc.properties" />
    	</bean>
    
    <!--配置数据源-->
    <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource"
          destroy-method="close">
        <property name="driverClassName" value="${jdbc.driver}" />
        <property name="url" value="${jdbc.url}" />
        <property name="username" value="${jdbc.username}" />
        <property name="password" value="${jdbc.password}" />
        <!-- 初始化连接大小 -->
        <property name="initialSize" value="${jdbc.initialSize}"></property>
        <!-- 连接池最大数量 -->
        <property name="maxActive" value="${jdbc.maxActive}"></property>
        <!-- 连接池最大空闲 -->
        <property name="maxIdle" value="${jdbc.maxIdle}"></property>
        <!-- 连接池最小空闲 -->
        <property name="minIdle" value="${jdbc.minIdle}"></property>
        <!-- 获取连接最大等待时间 -->
        <property name="maxWait" value="${jdbc.maxWait}"></property>
    </bean>
    ```

- 配置DAO接口所在包

  - 我们知道mybatis是使用接口和mapping文件映射的方式进行CRUD的，当然也可以直接注解啥的，但是一定会有一个DAO的包，所以，这个配置就是让spring可以发现它

    ```xml
    <!-- DAO接口所在包名，Spring会自动查找其下的类 -->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="InterfaceDao" />
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory" />	
    </bean>
    ```

- 配置mapping

  - 其实还是建议大家使用mapping的方式去写sql，所以这里需要再配置一下mapping的路径发现，以及其他的配置，相当于这是mybaits和spring的整合入口。

    ```xml
    <!-- spring和MyBatis完美整合，不需要mybatis的配置映射文件 -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource" />
        <!-- 自动扫描mapping.xml文件 -->
        <property name="mapperLocations" value="classpath:mapping/*.xml"/>
    </bean>
    ```

- 配置事务管理

  ```xml
  <!-- (事务管理)transaction manager, use JtaTransactionManager for global tx -->
  <bean id="transactionManager"
        class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
      <property name="dataSource" ref="dataSource" />
  </bean>
  ```


##### spring-shiro.xml

- sercurityManager

  ```xml
  <!-- securityManager -->
  <bean id ="securityManager" class ="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
      <!--  <property name ="realm" ref ="customRealm" />-->
     <!-- <property name ="cacheManager" ref ="cacheManager" /> -->
      <!--<property name ="sessionManager" ref ="sessionManager"/> -->
  </bean>
  ```


- lifecycle

  ```xml
  <bean id="lifecycleBeanPostProcessor" class="org.apache.shiro.spring.LifecycleBeanPostProcessor" />  
  ```


- shiro注解

  ```xml
  <bean id="defaultAdvisorAutoProxyCreator" class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator">  
    <!-- 必须改为true，即使用cglib方式为Action创建代理对象。默认值为false，使用JDK创建代理对象，会造成问题 -->  
    	<property name="proxyTargetClass" value="true"></property>  
  </bean>  
  ```


- shiroFilter

  ```xml
  <bean id ="shiroFilter" class = "org.apache.shiro.spring.web.ShiroFilterFactoryBean">
      <property name ="securityManager" ref ="securityManager" />
  <!-- loginUrl认证提交地址，如果没有认证将会请求此地址进行认证，请求地址将由formAuthenticationFilter进行表单认证 -->
      <property name ="loginUrl" value ="/user/login"/>
      <property name ="unauthorizedUrl" value ="/error"/>
  <!-- 过滤器链定义:从上向下顺序执行，一般将/**放在最下边 
  	<property name ="filters">
  		<map>
  			<entry key ="authc" value-ref ="formAuthenticationFilter"/>
  		</map>
  	</property>-->
      <property name ="filterChainDefinitions">
          <value>
          	<!-- 对静态资源设置匿名访问 -->
          	
          </value>
      </property>
  </bean>
  ```

  最后成功跑通:

  ![](C:\Users\A\Desktop\笔记\我的笔记_files\SSM+shiro6.png)

### 三、测试项目

#### 1、测试Spring框架

编写Controller和Service进行Hello world测试：

![](C:\Users\A\Desktop\笔记\我的笔记_files\SSM+shiro7.png)

- TestController.java

  ```java
  package cn.bestrivenlf.controller;
  
  import org.springframework.beans.factory.annotation.Autowired;
  import org.springframework.stereotype.Controller;
  import org.springframework.web.bind.annotation.RequestMapping;
  import org.springframework.web.bind.annotation.ResponseBody;
  
  import cn.bestrivenlf.service.SayHelloService;
  
  @Controller
  @RequestMapping("Test")
  public class TestController {
  	
  	@Autowired
  	private SayHelloService sayHelloService;
  	@RequestMapping("hello")
  	@ResponseBody
  	public String sayHello() {
  		return sayHelloService.sayHello();
  	}
  }
  
  ```


- SayHelloService.java

  ```java
  package cn.bestrivenlf.service;
  
  public interface SayHelloService {
  	public String sayHello();
  }
  
  ```


- SayHelloServiceImpl.java

  ```java
  package cn.bestrivenlf.serviceImpl;
  
  import org.springframework.stereotype.Service;
  
  import cn.bestrivenlf.service.SayHelloService;
  
  @Service
  public class SayHelloServiceImpl implements SayHelloService {
  
  	public String sayHello() {
  		// TODO Auto-generated method stub
  		return "hello world";
  	}
  
  }
  
  ```


项目启动后访问 localhost:8080/SSM-Shiro/Test/hello返回如下：

![](C:\Users\A\Desktop\笔记\我的笔记_files\SSM+shiro8.png)

#### 2、测试shiro和mybatis框架

​	详情见shiro笔记