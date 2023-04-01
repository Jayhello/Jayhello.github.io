# Filter

## 1 什么是 Filter

1. Filter 的基本功能是对 Servlet 容器调用 Servlet 的过程进行拦截，从而在 Servlet 进行响应处理的前后实现一些特殊的功能。
2. 在 Servlet API 中定义了三个接口类来开供开发人员编写 Filter 程序：Filter, FilterChain, FilterConfig
3. Filter 程序是一个实现了 Filter 接口的 Java 类，与 Servlet 程序相似，它由 Servlet 容器进行调用和执行
4. Filter 程序需要在 web.xml 文件中进行注册和设置它所能拦截的资源：Filter 程序可以拦截 Jsp, Servlet, 静态图片文件和静态 html 文件



## 2 如何创建并应用一个 Filter

1. 创建一个 Filter 类，实现 Filter  接口。

   ```java
   public class HelloFilter implements Filter
   ```

   

2. 在 web.xml 文件中配置并映射该 Filter，其中 url-pattern 指定该 Filter 可以拦截哪些资源，即可以通过哪些 url 访问到该 Filter。

   ```xml
   <!-- 注册filter -->
     <filter>
       <filter-name>helloFilter</filter-name>
       <filter-class>com.husthuangkai.filter.HelloFilter</filter-class>
     </filter>
     
     <filter-mapping>
       <filter-name>helloFilter</filter-name>
       <url-pattern>/hello.jsp</url-pattern>
     </filter-mapping>
   ```

3. \<dispatcher\> 子标签的意义

   - REQUEST：当用户直接访问页面时，Web容器将会调用过滤器。如果目标资源是通过RequestDispatcher的include()或forward()方法访问时，那么该过滤器就不会被调用。
   - INCLUDE：如果目标资源是通过RequestDispatcher的include()方法访问时，那么该过滤器将被调用。除此之外，该过滤器不会被调用。
   - FORWARD：如果目标资源是通过RequestDispatcher的forward()方法访问时，那么该过滤器将被调用，除此之外，该过滤器不会被调用。
   - ERROR：如果目标资源是通过声明式异常处理机制调用时，那么该过滤器将被调用。除此之外，过滤器不会被调用。



## 3 Filter 相关的 API

1. Filter 接口

   ```java
   public class HelloFilter implements Filter {
   
       /**
        * Filter 的逻辑代码，拦截请求并处理。
        * @param FilterChain 多个 Filter 可以构成一个 Filter 链。
        */
       @Override
       public void doFilter(ServletRequest arg0, ServletResponse arg1, FilterChain arg2)
               throws IOException, ServletException {
           System.out.println("doFilter");
           
           // 放过此请求，进入下一个 Filter
           arg2.doFilter(arg0, arg1);
       }
       
       // 在创建 Filter 对象后立即被调用，且只被调用一次。Filter 对象在加载当前 WEB 应用时即被创建
       // 该方法用于对当前 Filter 对象进行初始化操作
       // Filter 实例是单例的
       // @param filterConfig 类似于 servletConfig
       @Override
       public void init(FilterConfig filterConfig) throws ServletException {
           System.out.println("init");
       }
       
       // Filter 对象被销毁时调用，只被调用一次
       @Override
       public void destroy() {
           System.out.println("destory");
       }
   
   }
   ```

   多个 Filter  拦截的顺序和 filter-mapping 的顺序有关，先写的先拦截。

   