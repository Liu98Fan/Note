# SpringBoot的日志系统

## 1、日志框架	

​	Spring Boot 底层的日志框架采用了slf4j+logback实现，其中slf4j实现日志的抽象层，logback是日志的实现层，两者结合形成了spring Boot的日志框架。

## 2、使用

​	slf4j采用的也是工厂的设计模式

```java
public class Mydemo01ApplicationTests {
    //记录器
    Logger logger = LoggerFactory.getLogger(getClass());
    @Test
    public void contextLoads() {
        //日志的级别，由低到高分别是: trace<debug<info<warn<error
        //可以调整输出的日志级别，日志就只会输出这个级别及更高的界别生效
        //Spring Boot默认输出info级别的
        logger.trace("这是trace日志");
        logger.debug("这是debug日志");
        logger.info("这是info日志");
        logger.warn("这是warn日志");
        logger.error("这是error日志");
    }

}
```

​	可以在配置文件中修改日志的一些配置：

```properties
#####日志配置#########
#调整log级别
logging.level.cn.bestrivenlf=trace

#不指定路径则当前项目下生成springboot.log的日志
#也可以指定完成的路径
#logging.file=springboot.log

#这是和logging.file的冲突设置，一般只设置path，会在path下创建默认spring.log的日志文件
#在当前磁盘的根路径下创建spring文件夹和里面的log文件夹，然后使用spring.log作为默认的日志文件
logging.path=/spring/log
# 指定日志在控制台输出的日志格式
# logging.pattern.console=
# 指定文件种日志的输出格式
# logging.pattern.file=
```

## 3、特殊配置

​	在resource下可以存放日志的配置文件用以覆盖springBoot的默认日志配置，但是对于配置文件的文件名有固定要求，详情可以看官方文档。

​	对于logback的日志配置文件，可以使用logback.xml和logback-spring.xml作为配置文件名，但是springBoot官方文档建议使用带spring后缀的文件名，因为如果使用logback.xml则是直接被logback的日志框架识别，从而绕过springBoot，而使用logback-spring.xml作为配置文件名则是通过springBoot来修改日志的默认配置，且可以使用springBoot提供的特殊配置。

​	例如：不同场景下生效不同的配置

```xml
 <appender name="stdout" class="ch.qos.logback.core.ConsoleAppender">
        <!--
        日志输出格式：
			%d表示日期时间，
			%thread表示线程名，
			%-5level：级别从左显示5个字符宽度
			%logger{50} 表示logger名字最长50个字符，否则按照句点分割。 
			%msg：日志消息，
			%n是换行符
        -->
        <layout class="ch.qos.logback.classic.PatternLayout">
            <springProfile name="dev">
                <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS}----> [%thread]------> %-5level %logger{50} - %msg%n</pattern>
            </springProfile>
            <springProfile name="!dev">
                <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS}===== [%thread]======= %-5level %logger{50} - %msg%n</pattern>
            </springProfile>

        </layout>
    </appender>
```

## 4、切换日志框架

​	如果想要使用log4j的实现框架，则需要在springboot种排除掉logback的jar包，然后倒入slf4j对log4j的支持jar即可。

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <exclusions>
                <exclusion>
                    <artifactId>logback-classic</artifactId>
                    <groupId>ch.qos.logback</groupId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
        </dependency>
```

