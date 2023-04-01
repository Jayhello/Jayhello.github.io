# ServletContext

## 1 简介

1. Servlet引擎为每个WEB应用程序都创建一个对应的ServletContext对象，ServletContext对象被包含在ServletConfig对象中，调用ServletConfig.getServletContext方法可以返回ServletContext对象的引用。
2. 由于一个WEB应用程序中的所有Servlet都共享同一个ServletContext对象，所以，**ServletContext对象被称之为 application 对象（Web应用程序对象）**。  
3. 功能：
   - 获取WEB应用程序的初始化参数 
   - 记录日志 
   - application域范围的属性 
   - 访问资源文件 
   - 获取虚拟路径所映射的本地路径 
   - WEB应用程序之间的访问 
   - ServletContext的其他方法 

## 2 使用方法

1. 可以由ServletConfig对象获得

2. 该对象代表当前WEB应用：可以认为ServletContext时当前WEB应用的一个大管家，可以从中获取到当前WEB应用各个方面的信息。

   ServletContext的初始化参数可以被所有的Servlet获取，是全局的初始化参数。而Servlet的初始化参数是某一个Servlet的初始化参数，是局部的初始化参数。

   ```xml
   <!-- 配置当前WEB应用的初始化参数 -->
     <context-param>
     	<param-name>driver</param-name>
     	<param-value>com.mysql.jdbc.Driver</param-value>
     </context-param>
   ```

   ```java
   // 获取servletContext
   		ServletContext servletContext = servletConfig.getServletContext();
   		
   		String driver = servletContext.getInitParameter("driver");
   		System.out.println("-->" + driver);
   		
   		Enumeration<String> paramsEnumeration = servletContext.getInitParameterNames();
   		while (paramsEnumeration.hasMoreElements()) {
   			String paramNameString = paramsEnumeration.nextElement();
   			String valueString = servletContext.getInitParameter(paramNameString);
   			System.out.println("name: " + paramNameString + " value: " + valueString);
   		}
   ```

   > -->com.mysql.jdbc.Driver
   > name: driver value: com.mysql.jdbc.Driver

3. getRealPath：获取当前WEB应用的某一个文件在服务器上的路径，而不是部署前的路径。

   ```java
   System.out.println(servletContext.getRealPath("/hello.hsp"));
   ```

   > D:\HuangKaiPrivate\JavaWorkSpace\.metadata\.plugins\org.eclipse.wst.server.core\tmp0\wtpwebapps\secondWebApp\hello.hsp

4. getContextPath()：获取当前应用的名称

5. getResource获取当前WEB应用某一个文件对应的输入流名称

6. 和attrubute相关的几个方法