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

## 3 Servlet容器：运行Servlet、JSP、Filter等的软件环境

1. 可以来创建Servlet对象，并调用Servlet的相关生命周期方法。
2. JSP，Filter，Listener，Tag...



## 4 Servlet生命周期的方法

以下方法都是由servlet容器负责调用

1. 构造器：只有第一次请求servlet时，创建servlet的实例，调用构造器，这说明servlet是单实例的！既然是单实例的就要考虑线程同步问题。
2. init方法：只被调用一次，在创建好实例后立即被调用，用于初始化当前servlet。
3. service：被多次调用，每次请求都会调用service方法。
4. destory：只被调用一次，在当前servlet所在的WEB应用被卸载前调用，用于释放当前servlet所占用的资源。

## 5 load-on-startup参数

1. load-on-startup配置在servlet节点中：

   ```xml
     <!-- 配置和映射Servlet -->
     <servlet>
     	<!-- Servlet注册的名字 -->
     	<servlet-name>helloServlet</servlet-name>
     	<!-- Servlet的全类名 -->
     	<servlet-class>com.husthuangkai.javaweb.HelloServlet</servlet-class>
     	<!-- 可以指定servlet实例被创建的时机 -->
     	<load-on-startup>1</load-on-startup>
     </servlet>
   ```

2. load-on-startup：可以指定servlet被创建的时机，若为负数，则在第一次请求时被创建。若为0或正数，则在当前WEB应用被servlet容器加载时创建实例，且数组越小越早被创建。



## 6 Servlet容器响应客户请求的过程

1. Servlet引擎检查是否已经装载并创建了该Servlet的实例对象。如果是，则直接执行第④步，否则，执行第②步。
2. 装载并创建该Servlet的一个实例对象：调用该 Servlet 的构造器 
3. 调用Servlet实例对象的init()方法。
4. 创建一个用于封装请求的ServletRequest对象和一个代表响应消息的ServletResponse对象，然后调用Servlet的service()方法并将请求和响应对象作为参数传递进去。
5. WEB应用程序被停止或重新启动之前，Servlet引擎将卸载Servlet，并在卸载之前调用Servlet的destroy()方法。

## 7 Servlet的注册与运行 

1. Servlet程序必须通过Servlet容器来启动运行，并且储存目录有特殊要求，通需要存储在<WEB应用程序目录>\WEB-INF\classes\目录中。 
2. Servlet程序必须在WEB应用程序的web.xml文件中进行注册和映射其访问路径，才可以被Servlet引擎加载和被外界访问。
3. 一个<servlet>元素用于注册一个Servlet，它包含有两个主要的子元素：<servlet-name>和<servlet-class>，分别用于设置Servlet的注册名称和Servlet的完整类名。 
4. 一个<servlet-mapping>元素用于映射一个已注册的Servlet的一个对外访问路径，它包含有两个子元素：<servlet-name>和<url-pattern>，分别用于指定Servlet的注册名称和Servlet的对外访问路径。

## 8 Servlet映射的细节 

1. 同一个Servlet可以被映射到多个URL上，即多个<servlet-mapping>元素的<servlet-name>子元素的设置值可以是同一个Servlet的注册名。 
2. 在Servlet映射到的URL中也可以使用*通配符，但是只能有两种固定的格式：一种格式是“*.扩展名”，另一种格式是以正斜杠（/）开头并以“/*”结尾。