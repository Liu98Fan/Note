## Shiro权限认证

### 一、框架搭建

各种依赖见SSM+Shiro搭建笔记

### 二、登陆认证

#### 1、最简单的用户名+密码登陆

首先需要一个login.jsp页面用来登陆

```xml
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<meta http-equiv="X-UA-Compatible" content="IE=edge">
<meta name="viewport" content="width=device-width, initial-scale=1">

<title>标题</title>
</head>
<body>

<!--
	@Author:liufan
	@Date:2018年11月17日
	@Description:求无bug。
 -->	
 <form action="/SSM-Shiro/Test/login">
 	用户名:<input name="username" />
 	<br>
 	密码：<input name="password" />
 	<br>
 	<input type="submit" value="login"/>
 
 </form>
 
</body>
</html>
```

然后就是controller层写一个接收表单的handler：

```java
package cn.bestrivenlf.controller;
import org.apache.shiro.SecurityUtils;
import org.apache.shiro.authc.UsernamePasswordToken;
import org.apache.shiro.subject.Subject;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

import cn.bestrivenlf.service.SayHelloService;

@Controller
@RequestMapping("/Test")
public class TestController {
	
	@RequestMapping("/login")
	public String login(String username,String password) {
        //首先要获取到subject，通过SecurityUtils.getSubject方法。
		Subject subject = SecurityUtils.getSubject();
        //然后判断当前subject是否被认证过，如果没有则进行认证。
		System.out.println("认证开始");
		if(!subject.isAuthenticated()) {
            //认证开始后，需要将用户名和密码组装程一个token传入我们的自定义realm
			UsernamePasswordToken token = new UsernamePasswordToken(username,password);
			try {
                //调用subject的login方法将认证信息传入我们自己的realm方法进行认证
				subject.login(token);
			}catch (Exception e) {
				// TODO: handle exception
				System.out.println("fail authentication");
			}
			
		}
		return "index";
	}
}
```

接下来就是写我们自己的认证realm类了，需要继承AuthenticatingRealm这个类

```java
package cn.bestrivenlf.Realm;

import org.apache.shiro.authc.AuthenticationException;
import org.apache.shiro.authc.AuthenticationInfo;
import org.apache.shiro.authc.AuthenticationToken;
import org.apache.shiro.authc.SimpleAuthenticationInfo;
import org.apache.shiro.authc.UnknownAccountException;
import org.apache.shiro.authc.UsernamePasswordToken;
import org.apache.shiro.realm.AuthenticatingRealm;
import org.springframework.beans.factory.annotation.Autowired;

import cn.bestrivenlf.dao.UserDao;
import cn.bestrivenlf.entity.User;


public class MyRealm extends AuthenticatingRealm {

	@Autowired
	private UserDao userdao;
	@Override
	protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
		// TODO Auto-generated method stub
        //首先接收来自handler的token后需要先强转成UsernamePasswordToken
		UsernamePasswordToken upToken = (UsernamePasswordToken)token;
        //从token中获取登陆用户名
		String username = upToken.getUsername();
        //接下来应该从数据库中查找用户信息，这里模拟数据
		User user = new User();
        //模拟数据用户名的密码是123456
		user.setUsername(username);
		user.setPassword("123456");
        //这里封装用户信息交给Shiro去比对
        /**
        没错我们自己不需要比对信息，将用户提交的信息和数据库查找的信息交给Shiro。Shiro会以一种更为安全的方式进行信息的比对并返回结果。
        **/
        //封装用户身份信息，即当前username或者user实体类
		Object principal = user;
        //传入这个用户的正确密码
		String credential = user.getPassword();
        //realm的name，调用父类的getName方法即可
		String realmName = super.getName();
        //封装成一个SimpleAuthenticationInfo，将其返回给shiro即可
		SimpleAuthenticationInfo info = new SimpleAuthenticationInfo(principal,credential,realmName);
		return info;
		
	}
	
}

```

如果仅仅是简单的登陆信息认证，到这里就结束了，当然还需要将我们的Realm进行配置，当然是配置在SercurityManager里面了：

```xml
<bean id ="securityManager" class ="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
      <property name ="realm" ref ="customRealm" />
   <!-- <property name ="cacheManager" ref ="cacheManager" /> -->
    <!--<property name ="sessionManager" ref ="sessionManager"/> -->
</bean>

<!-- realm -->
<bean id ="customRealm" class ="cn.bestrivenlf.Realm.MyRealm"/>
```

然后在认证链中将所有请求都更改为authc拦截器，即所有路径都要认证，当然登陆请求还是要放开的：

```xml
<bean id ="shiroFilter" class = "org.apache.shiro.spring.web.ShiroFilterFactoryBean">
    <property name ="securityManager" ref ="securityManager" />
<!-- loginUrl认证提交地址，如果没有认证将会请求此地址进行认证，请求地址将由formAuthenticationFilter进行表单认证 -->
    <property name ="loginUrl" value ="/login.jsp"/>
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
        	/login.jsp=anon
        	/user/logout = logout
        	/Test/login=anon
        	/**=authc
        </value>
    </property>
</bean>
```

至此，简单的登陆验证就完成了。

#### 2、密码加密验证

首先选择我们的密码加密方式，比如MD5加密，然后配置在我们的Realm上：

```xml
<bean id ="customRealm" class ="cn.bestrivenlf.Realm.MyRealm">
<property name="credentialsMatcher">
	<bean class="org.apache.shiro.authc.credential.HashedCredentialsMatcher">
        <!--使用的加密算法-->
		<property name="hashAlgorithmName" value="MD5">
		</property>
        <!--加密次数-->
        <property name="hashIterations" value="1000"></property>
	</bean>
</property>
</bean>
```

然后realm中的代码稍作修改，加入盐值和修改构造函数即可：

```java
	protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
		// TODO Auto-generated method stub
		UsernamePasswordToken upToken = (UsernamePasswordToken)token;
		String username = upToken.getUsername();
		User user = new User();
		user.setUsername(username);
		user.setPassword("16fe90cb7a2c8a18faf0adcb74e92dbe");
		ByteSource salt = ByteSource.Util.bytes("shagou");
		Object principal = user;
		String credential = user.getPassword();
		String realmName = super.getName();
		SimpleAuthenticationInfo info = new SimpleAuthenticationInfo(principal,credential,salt,realmName);
		return info;
		
	}
```

### 三、登陆授权

讲完认证，接下来就是授权操作。

之前认证的时候我们讲到自定义Realm要继承AuthenticatingRealm来实现它的抽象方法，授权的时候也要继承一个授权类，可是java无法多继承，所以可以直接继承AuthorzingRealm，它包含了一个授权一个认证的抽象方法。

![](C:\Users\LIUFAN\Desktop\笔记\Note\我的笔记_files\SSM+shiro9.png)

接下来就进行授权：

其实就是多实现一个方法

```java
protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
		// TODO Auto-generated method stub
		User principal = (User)principals.getPrimaryPrincipal();
		String username = principal.getUsername();
		Set<String> roles = new HashSet<String>();
		if(username.equals("shagou")){
			roles.add("user");
		}else if (username.equals("admin")){
			roles.add("user");
			roles.add("admin");
		}
		return new SimpleAuthorizationInfo(roles);
	}
```

这里分别设置了有user和admin两个角色，对应shagou和admin两个账号，密码都是123456。

前端页面使用相应的shiro标签：

注意引入命名空间：

```html
<%@ taglib prefix="shiro" uri="http://shiro.apache.org/tags"%>
```

#### 使用标签

```html
<html>
<%@ taglib prefix="shiro" uri="http://shiro.apache.org/tags"%>
<body>
<h2>Hello World!</h2>
<shiro:hasRole name="user"><a href="/SSM-Shiro/Test/user">user.jsp</a></shiro:hasRole>
<br>
<shiro:hasRole name="admin"><a href="/SSM-Shiro/Test/admin">admin.jsp</a></shiro:hasRole>
<a href="/SSM-Shiro/user/logout">logout</a>
</body>
</html>

```

至此，一个完整的授权过程就结束了

#### 使用注解

也可以在方法前面使用相应的注解来进行权限控制

```java
	@RequiresRoles({"admin"})
	@RequestMapping("/hello")
	
	public String sayHello() {
		Subject su = SecurityUtils.getSubject();
		
		return "admin";
	}
```



### 四、一些问题：

#### 1、注解失效

不管怎么写权限注解都能进入方法，简单说就是shiro并没有把没有权限的用户拦截下来，那么就要考虑是否是shiro的配置文件出现了问题。必要注解是以下：

```xml
<!--shiro的配置文件-->
<!-- lifecycle -->
<bean id="lifecycleBeanPostProcessor" class="org.apache.shiro.spring.LifecycleBeanPostProcessor" />  

<!-- 开启shiro的注解支持 -->  
<bean id="defaultAdvisorAutoProxyCreator" class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator">  
  <!-- 必须改为true，即使用cglib方式为Action创建代理对象。默认值为false，使用JDK创建代理对象，会造成问题 -->  
  	<property name="proxyTargetClass" value="true"></property>  
</bean>  
 <bean class="org.apache.shiro.spring.security.interceptor.AuthorizationAttributeSourceAdvisor"></bean> 
```

```xml
<!--spring的配置文件-->
<bean class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator"
        depends-on="lifecycleBeanPostProcessor" />
<bean class="org.apache.shiro.spring.security.interceptor.AuthorizationAttributeSourceAdvisor">
        <property name="securityManager" ref="securityManager" />
</bean>
```

