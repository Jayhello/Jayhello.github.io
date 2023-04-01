### 1 SpringBoot是如何运行在Tomcat上的

我们知道，最早在使用原生的Servlet编写web应用程序的时候，需要有一个web.xml文件，在web.xml文件中需要配置Servlet的类名以及其响应的路径。那么问题来了，在SpringBoot中并没有web.xml文件，那么SpringBoot是如何运行在Tomcat上的呢。

1. 根据Servlet3.0的新规范，Tomcat在启动时，会首先运行继承了ServletContainerInitializer接口的

   onStartup方法，对于SpringBoot而言，就是这个类。

   ```java
   org.springframework.web.SpringServletContainerInitializer;
   
   @HandlesTypes(WebApplicationInitializer.class)
   public class SpringServletContainerInitializer implements ServletContainerInitializer {
   	@Override
   	public void onStartup(@Nullable Set<Class<?>> webAppInitializerClasses, ServletContext servletContext)
   			throws ServletException {
   	//......
   	}
   
   }
       
   ```

2. 然后，它会将所有继承了下面这个注解中传入的这个接口的类，作为参数传给onStartup方法。

   ```java
   @HandlesTypes(WebApplicationInitializer.class)
   ```

3. 然后，他会在onStartup方法中，遍历所有的webAppInitializerClasse类，并调用其onStartup方法。

   ```java
   	public void onStartup(@Nullable Set<Class<?>> webAppInitializerClasses, ServletContext servletContext)
   			throws ServletException {
   	//......
           for (WebApplicationInitializer initializer : initializers) {
   			initializer.onStartup(servletContext);
   		}
   	}
   ```

   