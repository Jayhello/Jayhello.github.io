# Servlet

## 1 Servlet简介

1. Java Servlet是和平台无关的服务器端组件，它运行在Servlet容器中。Servlet容器负责Servlet和客户的通信以及调用Servlet的方法，Servlet和客户的通信采用“请求/响应”的模式。
2. Servlet可完成如下功能：
   - 创建并返回基于客户请求的动态HTML页面。
   - 创建可嵌入到现有HTML 页面中的部分HTML 页面（HTML 片段）。
   - 与其它服务器资源（如数据库或基于Java的应用程序）进行通信。



## 2 Servlet的HelloWorld

1. 创建一个Servlet接口的实现类

   ```java
   public class HelloServlet implements Servlet {
   
   	@Override
   	public void destroy() {
   		// TODO Auto-generated method stub
   		System.out.println("destory");
   	}
   
   	@Override
   	public ServletConfig getServletConfig() {
   		// TODO Auto-generated method stub
   		System.out.println("getServletConfig");
   		return null;
   	}
   
   	@Override
   	public String getServletInfo() {
   		// TODO Auto-generated method stub
   		System.out.println("getServletInfo");
   		return null;
   	}
   
   	@Override
   	public void init(ServletConfig arg0) throws ServletException {
   		System.out.println("init");
   		// TODO Auto-generated method stub
   		
   	}
   
   	@Override
   	public void service(ServletRequest arg0, ServletResponse arg1) throws ServletException, IOException {
   		// TODO Auto-generated method stub
   		System.out.println("service");
   		
   	}
   	
   	public HelloServlet() {
   		System.out.println("HelloServlet constructor");
   	}
   
   }
   
   ```

2. 在web.xml文件中映射这个Servlet

   ```xml
     <!-- 配置和映射Servlet -->
     <servlet>
     	<servlet-name>helloServlet</servlet-name>
     	<servlet-class>com.husthuangkai.javaweb.HelloServlet</servlet-class>
     </servlet>
     
     <servlet-mapping>
     	<servlet-name>helloServlet</servlet-name>
     	<url-pattern>/hello</url-pattern>
     </servlet-mapping>
     
   ```

3. 访问

   > http://localhost:8080/secondWebApp/hello