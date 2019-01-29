# 1.servlet3

## 1.1 无web.xml文件，启动web项目

Shared libraries（共享库） / runtimes pluggability（运行时插件能力）

1、Servlet容器启动会扫描，当前应用里面每一个jar包的ServletContainerInitializer的实现
2、提供ServletContainerInitializer的实现类；
	必须绑定在，META-INF/services/javax.servlet.ServletContainerInitializer
	文件的内容就是ServletContainerInitializer实现类的全类名；

总结：容器在启动应用的时候，会扫描当前应用每一个jar包里面
META-INF/services/javax.servlet.ServletContainerInitializer
指定的实现类，启动并运行这个实现类的方法；传入感兴趣的类型；

- javax.servlet.ServletContainerInitializer:

```
com.nyf.servlet.MyServletContainerInitializer
```

- MyServletContainerInitializer.java

```java
package com.nyf.servlet;
//容器启动的时候会将@HandlesTypes指定的这个类型下面的子类（实现类，子接口等）传递过来；
//传入感兴趣的类型；
@HandlesTypes(value={HelloService.class})
public class MyServletContainerInitializer implements ServletContainerInitializer {
	/**
	 * 应用启动的时候，会运行onStartup方法；
	 * 
	 * Set<Class<?>> arg0：感兴趣的类型的所有子类型；
	 * ServletContext arg1:代表当前Web应用的ServletContext；一个Web应用一个ServletContext；
	 * 
	 * 1）、使用ServletContext注册Web组件（Servlet、Filter、Listener）
	 * 2）、使用编码的方式，在项目启动的时候给ServletContext里面添加组件；
	 * 		必须在项目启动的时候来添加；
	 * 		1）、ServletContainerInitializer得到的ServletContext；
	 * 		2）、ServletContextListener得到的ServletContext；
	 */
	@Override
	public void onStartup(Set<Class<?>> arg0, ServletContext sc) throws ServletException {
		System.out.println("感兴趣的类型：");
		for (Class<?> claz : arg0) {
			System.out.println(claz);
		}
		
		//注册组件  ServletRegistration  
		ServletRegistration.Dynamic servlet = sc.addServlet("userServlet", new UserServlet());
		//配置servlet的映射信息
		servlet.addMapping("/user");
		
		//注册Listener
		sc.addListener(UserListener.class);
		
		//注册Filter  FilterRegistration
		FilterRegistration.Dynamic filter = sc.addFilter("userFilter", UserFilter.class);
		//配置Filter的映射信息
		filter.addMappingForUrlPatterns(EnumSet.of(DispatcherType.REQUEST), true, "/*");
	}
}
```

- HelloService系列

```java
public interface HelloService {}
public abstract class AbstractHelloService implements HelloService {}
public interface HelloServiceExt extends HelloService {}
public class HelloServiceImpl implements HelloService {}
```

- UserServlet.java

```java
public class UserServlet extends HttpServlet {
	@Override
	protected void doGet(HttpServletRequest req, HttpServletResponse resp) 
        		throws ServletException, IOException {
		resp.getWriter().write("tomcat...");
	}
}
```

- UserListener.java

```java
/** 监听项目的启动和停止 */
public class UserListener implements ServletContextListener {
	//监听ServletContext销毁
	@Override
	public void contextDestroyed(ServletContextEvent arg0) {
		System.out.println("UserListener...contextDestroyed...");
	}
	//监听ServletContext启动初始化
	@Override
	public void contextInitialized(ServletContextEvent arg0) {
		ServletContext servletContext = arg0.getServletContext();
		System.out.println("UserListener...contextInitialized...");
	}
}
```

- UserFilter.java

```java
public class UserFilter implements Filter {
	@Override
	public void destroy() {}
	@Override
	public void doFilter(ServletRequest arg0, ServletResponse arg1, FilterChain arg2)
			throws IOException, ServletException {
		// 过滤请求
		System.out.println("UserFilter...doFilter...");
		//放行
		arg2.doFilter(arg0, arg1);
	}
	@Override
	public void init(FilterConfig arg0) throws ServletException {}
}
```

## 1.2 异步处理请求

HelloServlet.java

```java
@WebServlet("/hello")
public class HelloServlet extends HttpServlet {
	@Override
	protected void doGet(HttpServletRequest req, HttpServletResponse resp) 
        		throws ServletException, IOException {
		System.out.println(Thread.currentThread()+" start...");
		try {
			sayHello();
		} catch (Exception e) {
			e.printStackTrace();
		}
		resp.getWriter().write("hello...");
		System.out.println(Thread.currentThread()+" end...");
	}
	
	public void sayHello() throws Exception{
		System.out.println(Thread.currentThread()+" processing...");
		Thread.sleep(3000);
	}
}
```

HelloAsyncServlet.java

```java
@WebServlet(value="/async",asyncSupported=true)  //打开异步处理请求
public class HelloAsyncServlet extends HttpServlet {
	
	@Override
	protected void doGet(HttpServletRequest req, HttpServletResponse resp) 
        		throws ServletException, IOException {
		//1、支持异步处理asyncSupported=true
		//2、开启异步模式
		System.out.println("主线程开始。。。"
                           +Thread.currentThread()+"==>"+System.currentTimeMillis());
		AsyncContext startAsync = req.startAsync();
		
		//3、业务逻辑进行异步处理;开始异步处理
		startAsync.start(new Runnable() {
			@Override
			public void run() {
				try {
					System.out.println("副线程开始。。。"
                     	 	+Thread.currentThread()+"==>"+System.currentTimeMillis());
					sayHello();
					startAsync.complete();
					//获取到异步上下文
					AsyncContext asyncContext = req.getAsyncContext();
					//4、获取响应
					ServletResponse response = asyncContext.getResponse();
					response.getWriter().write("hello async...");
					System.out.println("副线程结束。。。"
				          +Thread.currentThread()+"==>"+System.currentTimeMillis());
				} catch (Exception e) {
				}
			}
		});		
		System.out.println("主线程结束。。。"
                           +Thread.currentThread()+"==>"+System.currentTimeMillis());
	}

	public void sayHello() throws Exception{
		System.out.println(Thread.currentThread()+" processing...");
		Thread.sleep(3000);
	}
}
```
