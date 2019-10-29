# ServletConfig

ServletConfig封装了Servlet的配置信息，并且可以获取ServletContext对象

1. 配置Servlet的初始化参数

   ```xml
   <!-- 配置和映射Servlet -->
     <servlet>
     	<!-- Servlet注册的名字 -->
     	<servlet-name>helloServlet</servlet-name>
     	<!-- Servlet的全类名 -->
     	<servlet-class>com.husthuangkai.javaweb.HelloServlet</servlet-class>
     	
     	<!-- 配置servlet的初始化参数 ,且该节点必须在load-on-startup节点前面-->
     	<init-param>
     		<param-name>user</param-name>
     		<param-value>root</param-value>
     	</init-param>
     	
     	<init-param>
     		<param-name>password</param-name>
     		<param-value>123</param-value>
     	</init-param>
     	
     	<!-- 可以指定servlet实例被创建的时机 -->
     	<load-on-startup>1</load-on-startup>
     </servlet>
   ```

2. 获取初始化参数：

   getInitParameter(String name)：获取指定参数名的初始化参数

   getInitParameterNames()：获取参数名组成的Enumeration对象

   ```java
   	public void init(ServletConfig servletConfig) throws ServletException {
   		System.out.println("init");
   		
   		String user = servletConfig.getInitParameter("user");
   		System.out.println("user:" + user); 
   		
   		Enumeration<String> names = servletConfig.getInitParameterNames();
   		while (names.hasMoreElements()) {
   			String name = names.nextElement();
   			String value =servletConfig.getInitParameter(name);
   			System.out.println("name: " + name + " " + value);
   		}
   	}
   
   ```

   > 输出：
   >
   > init
   > user:root
   > name: password 123
   > name: user root