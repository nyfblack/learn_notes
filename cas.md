# 1.SSO

​	SSO单点登录全称Single Sign On，是指在多系统应用群中登录一个系统，便可在其他所有系统中得到授权而无需再次登录，包括单点登录与单点注销两部分;

## 1.登录

​	sso需要一个独立的认证中心，只有认证中心能接受用户的用户名密码等安全信息，其他系统不提供登录入口，只接受认证中心的间接授权。间接授权通过令牌实现，sso认证中心验证用户的用户名密码没问题，创建授权令牌，在接下来的跳转过程中，授权令牌作为参数发送给各个子系统，子系统拿到令牌，即得到了授权，可以借此创建局部会话，局部会话登录方式与单系统的登录方式相同。

1. 用户访问系统1的受保护资源，系统1发现用户未登录，跳转至sso认证中心，并将自己的地址作为参数
2. sso认证中心发现用户未登录，将用户引导至登录页面
3. 用户输入用户名密码提交登录申请
4. sso认证中心校验用户信息，创建用户与sso认证中心之间的会话，称为全局会话，同时创建授权令牌
5. sso认证中心带着令牌跳转会最初的请求地址（系统1）
6. 系统1拿到令牌，去sso认证中心校验令牌是否有效
7. sso认证中心校验令牌，返回有效，注册系统1
8. 系统1使用该令牌创建与用户的会话，称为局部会话，返回受保护资源
9. 用户访问系统2的受保护资源
10. 系统2发现用户未登录，跳转至sso认证中心，并将自己的地址作为参数
11. sso认证中心发现用户已登录，跳转回系统2的地址，并附上令牌
12. 系统2拿到令牌，去sso认证中心校验令牌是否有效
13. sso认证中心校验令牌，返回有效，注册系统2
14. 系统2使用该令牌创建与用户的局部会话，返回受保护资源



​	用户登录成功之后，会与sso认证中心及各个子系统建立会话，用户与sso认证中心建立的会话称为全局会话，用户与各个子系统建立的会话称为局部会话，局部会话建立之后，用户访问子系统受保护资源将不再通过sso认证中心，全局会话与局部会话有如下约束关系 

1. 局部会话存在，全局会话一定存在
2. 全局会话存在，局部会话不一定存在
3. 全局会话销毁，局部会话必须销毁



## 2.注销

​	单点注销，在一个子系统中注销，所有子系统的会话都将被销毁；sso认证中心一直监听全局会话的状态，一旦全局会话销毁，监听器将通知所有注册系统执行注销操作 

1. 用户向系统1发起注销请求
2. 系统1根据用户与系统1建立的会话id拿到令牌，向sso认证中心发起注销请求
3. sso认证中心校验令牌有效，销毁全局会话，同时取出所有用此令牌注册的系统地址
4. sso认证中心向所有注册系统发起注销请求
5. 各注册系统接收sso认证中心的注销请求，销毁局部会话
6. sso认证中心引导用户至登录页面



## 3.实现

[一个简单的sso案例](https://github.com/sheefee/simple-sso) 

sso采用客户端/服务端架构，sso-client与sso-server要实现的功能：

- sso-client

1. 拦截子系统未登录用户请求，跳转至sso认证中心
2. 接收并存储sso认证中心发送的令牌
3. 与sso-server通信，校验令牌的有效性
4. 建立局部会话
5. 拦截用户注销请求，向sso认证中心发送注销请求
6. 接收sso认证中心发出的注销请求，销毁局部会话

- sso-server

1. 验证用户的登录信息
2. 创建全局会话
3. 创建授权令牌
4. 与sso-client通信发送令牌
5. 校验sso-client令牌有效性
6. 系统注册
7. 接收sso-client注销请求，注销所有会话



# 2.CAS

## 1.CAS-server

- [安装包下载](https://github.com/apereo/cas-overlay-template/tree/5.3)

- 运行命令打成war包（需要先配置maven的环境变量）

  ```shell
  build.cmd package
  ```

- 把打包好的cas.war部署到tomcat上，启动tomcat；可以在浏览器中访问cas-server；

- 修改用户名和密码

  ```properties
  #打开文件tomcatDir\webapps\cas\WEB-INF\classes\application.properties
  #可以找到如下配置用户名和密码；此配置是users，意味着可以配置多个用户名密码；
  cas.authn.accept.users=casuser::Mellon
  #在此处添加,admin用户的用户名和密码
  cas.authn.accept.users=admin::admin
  ```

- 修改登录端口；只要修改tomcat的server.xml中的访问端口就可以（4.2以下的版本需要修改tomcat的访问端口和cas的配置端口）；由于https协议默认使用的端口为8443,我们修改为tomcat的8080端口 ，

  ```properties
  #在配置文件application.properties中修改：
  server.port=8080
  ```

- 由于CAS默认使用的是基于https协议,需要改为兼容使用http协议;

  - 5.3的配置文件application.properties ;

  ```properties
  #添加如下配置
  cas.tgc.secure=false
  cas.serviceRegistry.initFromJson=true
  ```

  - 兼容http: 

    ```json
    //打开文件tomcatDir\webapps\cas\WEB-INF\classes\services\HTTPSandIMAPS-10000001.json修改
    "serviceId" : "^(https|http|imaps)://.*",
    ```


## 2.关键配置的说明

  ```properties
  #application.properties
  #cas请求路径和ssl配置
  server.context-path=/cas
  server.port=8443
  # 这里路径指项目所在磁盘根目录  E:\etc\cas\bisa.keystore
  server.ssl.keyStore=/etc/cas/casServer.keystore
  server.ssl.keyStorePassword=test123
  server.ssl.keyPassword=test123
  server.ssl.enabled=true
  server.ssl.key-alias=cas.server.com
  ##
  # CAS Authentication Credentials
  # 默认静态认证，登录名为admin 密码为admin
  cas.authn.accept.users=admin::admin
  ###############################################################################
  # redis配置，将ticket票据存在redis中，默认在内存里
  #
  cas.ticket.registry.redis.host=192.168.1.12
  cas.ticket.registry.redis.database=1
  cas.ticket.registry.redis.port=8379
  cas.ticket.registry.redis.password=123456
  cas.ticket.registry.redis.timeout=2000
  cas.ticket.registry.redis.useSsl=false
  #不设置redis线程池
  cas.ticket.registry.redis.usePool=false
  ###############################################################################
  #数据库用户名
  cas.serviceRegistry.jpa.user=root
  #数据库密码
  cas.serviceRegistry.jpa.password=123456
  #mysql驱动
  cas.serviceRegistry.jpa.driverClass=com.mysql.jdbc.Driver
  #数据库连接
  cas.serviceRegistry.jpa.url=jdbc:mysql://127.0.0.1:3306/testshiro?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&useSSL=false
  cas.serviceRegistry.dialect=org.hibernate.dialect.MySQLDialect
  #连接池配置
  cas.serviceRegistry.jpa.pool.suspension=false
  cas.serviceRegistry.jpa.pool.minSize=6
  cas.serviceRegistry.jpa.pool.maxSize=18
  cas.serviceRegistry.jpa.pool.maxWait=2000
  cas.serviceRegistry.jpa.pool.timeoutMillis=1000
  #默认为create-drop，表示每次启动服务都会清除你之前注册的cas服务
  cas.serviceRegistry.jpa.ddlAuto=create-drop
  ################################################################################
  #配置单点登出
  #配置允许登出后跳转到指定页面
  cas.logout.followServiceRedirects=false
  #跳转到指定页面需要的参数名为 service
  cas.logout.redirectParameter=service
  #登出后需要跳转到的地址,如果配置该参数,service将无效。
  cas.logout.redirectUrl=https://www.taobao.com
  #在退出时是否需要 确认退出提示   true弹出确认提示框  false直接退出
  cas.logout.confirmLogout=true
  #是否移除子系统的票据
  cas.logout.removeDescendantTickets=true
  #禁用单点登出,默认是false不禁止
  #cas.slo.disabled=true
  #默认异步通知客户端,清除session
  #cas.slo.asynchronous=true
  ```

  

# 3.CAS-client

## 1.普通web项目

[cas-client-demo](https://blog.csdn.net/qq_24708791/article/details/78535565)

- pom.xml

```xml
<dependencies>
    <!-- cas -->  
    <dependency>  
        <groupId>org.jasig.cas.client</groupId>  
        <artifactId>cas-client-core</artifactId>  
        <version>3.3.3</version>  
    </dependency>       
</dependencies>  
```

- web.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns="http://java.sun.com/xml/ns/javaee"
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
    version="2.5">  
    <!-- 用于单点退出，该过滤器用于实现单点登出功能，可选配置 -->  
    <listener>  
     <listener-class>org.jasig.cas.client.session.SingleSignOutHttpSessionListener</listener-class>  
    </listener>  
    <!-- 该过滤器用于实现单点登出功能，可选配置。 -->  
    <filter>  
        <filter-name>CAS Single Sign Out Filter</filter-name>  
       <filter-class>org.jasig.cas.client.session.SingleSignOutFilter</filter-class>  
    </filter>  
    <filter-mapping>  
        <filter-name>CAS Single Sign Out Filter</filter-name>  
        <url-pattern>/*</url-pattern>  
    </filter-mapping>  
    <!-- 该过滤器负责用户的认证工作，必须启用它 -->  
    <filter>  
        <filter-name>CASFilter</filter-name>       
        <filter-class>org.jasig.cas.client.authentication.AuthenticationFilter</filter-class>  
        <init-param>  
            <param-name>casServerLoginUrl</param-name>  
            <param-value>http://localhost:9100/cas/login</param-value>  
            <!--这里的server是服务端的IP -->  
        </init-param>  
        <init-param>  
            <param-name>serverName</param-name>  
            <param-value>http://localhost:9001</param-value>
        </init-param>  
    </filter>  
    <filter-mapping>  
        <filter-name>CASFilter</filter-name>  
        <url-pattern>/*</url-pattern>  
    </filter-mapping>  
    <!-- 该过滤器负责对Ticket的校验工作，必须启用它 -->  
    <filter>  
        <filter-name>CAS Validation Filter</filter-name>  
        <filter-class>    org.jasig.cas.client.validation.Cas20ProxyReceivingTicketValidationFilter</filter-class>  
        <init-param>  
            <param-name>casServerUrlPrefix</param-name>  
            <param-value>http://localhost:9100/cas</param-value>  
        </init-param>  
        <init-param>  
            <param-name>serverName</param-name>  
            <param-value>http://localhost:9001</param-value>
        </init-param>  
    </filter>  
    <filter-mapping>  
        <filter-name>CAS Validation Filter</filter-name>  
        <url-pattern>/*</url-pattern>  
    </filter-mapping>  
    <!-- 该过滤器负责实现HttpServletRequest请求的包裹， 比如允许开发者通过HttpServletRequest的getRemoteUser()方法获得SSO登录用户的登录名，可选配置。 -->  
    <filter>  
        <filter-name>CAS HttpServletRequest Wrapper Filter</filter-name>  
        <filter-class>  
            org.jasig.cas.client.util.HttpServletRequestWrapperFilter</filter-class>  
    </filter>  
    <filter-mapping>  
        <filter-name>CAS HttpServletRequest Wrapper Filter</filter-name>  
        <url-pattern>/*</url-pattern>  
    </filter-mapping>  
    <!-- 该过滤器使得开发者可以通过org.jasig.cas.client.util.AssertionHolder来获取用户的登录名。 比如AssertionHolder.getAssertion().getPrincipal().getName()。 -->  
    <filter>  
        <filter-name>CAS Assertion Thread Local Filter</filter-name>       
        <filter-class>org.jasig.cas.client.util.AssertionThreadLocalFilter</filter-class>  
    </filter>  
    <filter-mapping>  
        <filter-name>CAS Assertion Thread Local Filter</filter-name>  
        <url-pattern>/*</url-pattern>  
    </filter-mapping>  
</web-app>
```

- index.jsp

```jsp
<%@ page language="java" contentType="text/html; charset=utf-8"
    pageEncoding="utf-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<title>test1</title>
</head>
    <body>
        欢迎登录test1
        <%=request.getRemoteUser()%>
    </body>
</html>
```



## 2.CAS与SpringBoot的集成

1. 在SpringBoot项目中添加依赖

   ```xml
   <!-- https://mvnrepository.com/artifact/net.unicon.cas/cas-client-autoconfig-support -->
   <dependency>
       <groupId>net.unicon.cas</groupId>
       <artifactId>cas-client-autoconfig-support</artifactId>
       <version>1.7.0-GA</version>
   </dependency>
   ```

2. 在SpringBoot项目的配置文件application.yml中添加CAS相关配置

   ```yaml
   cas:
     #cas服务端前缀,不是登录地址
     server-url-prefix: http://localhost:9100/cas
     #cas的登录地址
     server-login-url: http://localhost:9100/cas/login
     #当前客户端的地址
     client-host-url: http://localhost:9001
     #Ticket校验器使用CasProxyReceivingTicketValidationFilter
     #cas.validation-type目前支持3中方式：1、CAS；2、CAS3；3、SAML
     validation-type: cas
   
   server:
     port: 9001
   ```

3. 启动CAS

   ```JAVA
   @EnableCasClient   //开启casClient
   @SpringBootApplication
   public class CasStarterApplication {
       public static void main(String[] args) {
           SpringApplication.run(CasStarterApplication.class, args);
       }
   }
   ```

4. 编写一个restController测试

   ```java
   @RestController
   public class TestController {
       @GetMapping("/test")
       public String get(){
           return "rest success";
       }
   }
   ```

   







# 参考资料

- [使用配置资料](https://blog.csdn.net/u011872945/article/details/81047025)
- [CAS5.3.2单点登录](https://blog.csdn.net/qq_34021712/column/info/26952)
- [SpringBoot集成CAS5.3](https://blog.csdn.net/lhc0512/article/details/82466246)
- [cas5.2.x单点登录github](https://github.com/Sunhg2017/CAS_SSO_Record)









