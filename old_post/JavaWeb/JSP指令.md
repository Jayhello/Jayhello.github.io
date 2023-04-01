# JSP指令

## 1 简介

1. JSP指令（directive）是为JSP引擎而设计的，它们并不直接产生任何可见输出，而只是告诉引擎如何处理JSP页面中的其余部分。
2. JSP指令的基本语法格式：
   	<%@ 指令 属性名="值" %>
   	举例：<%@ page contentType="text/html;charset=gb2312"%>
   	注意：属性名部分是大小写敏感的
3. 在目前的JSP 2.0中，定义了page、include和taglib这三种指令，每种指令中又都定义了一些各自的属性。
   如果要在一个JSP页面中设置同一条指令的多个属性，可以使用多条指令语句单独设置每个属性，也可以使用同一条指令语句设置该指令的多个属性。 
   	第一种方式：
   		<%@ page contentType="text/html;charset=gb2312"%>
   		<%@ page import="java.util.Date"%>
   	第二种方式：
   		<%@ page contentType="text/html;charset=gb2312" import="java.util.Date"%> 



## 2 page指令

1. page指令用于定义JSP页面的各种属性，无论page指令出现在JSP页面中的什么地方，它作用的都是整个JSP页面，为了保持程序的可读性和遵循良好的编程习惯，page指令最好是放在整个JSP页面的起始位置。 

2. JSP 2.0规范中定义的page指令的完整语法：

   ```jsp
   <%@ page 
   	[ language="java" ] 
   	[ extends="package.class" ] 
   	[ import="{package.class | package.*}, ..." ] 
   	[ session="true | false" ] 
   	[ buffer="none | 8kb | sizekb" ] 
   	[ autoFlush="true | false" ] 
   	[ isThreadSafe="true | false" ] 
   	[ info="text" ] 
   	[ errorPage="relative_url" ] 
   	[ isErrorPage="true | false" ] 
   	[ contentType="mimeType [ ;charset=characterSet ]" | "text/html ; charset=ISO-8859-1" ] 
   	[ pageEncoding="characterSet | ISO-8859-1" ] 
   	[ isELIgnored="true | false" ] 
   %>
   
   ```

   常用的有：

   - import：指定要导哪些包。

   - session：取值为true或false，指定当前页面的session对象是否可用。

   - errorPage 和 isErrorPage：

     - errorPage指定若当前页面出现错误的实际响应页面是什么。其中 / 表示的是当前WEB应用的根目录。
     - 在响应 error.jsp 时，JSP 引擎使用请求转发的方式访问错误页面。

     - isErrorPage指定当前页面是否为错误处理页面，可以说明当前页面是否可以使用exception隐藏变量。需要注意的是：若指定isErrorPage=“true”，并使用exception的方法了，一般不建议能够直接访问该页面。

     - 如何使客户不直接访问页面呢？对于 Tomcat 服务器而言，WEB-INF 下的文件时不能够通过在浏览器中直接输入地址来访问的，但通过请求的转发是可以的。
     - 还可以在 web.xml 文件中配置错误页面。

   - contentType：指定当前 JSP 页面的相应类型。

     通常情况下，对于 JSP 页面而言，其取值均为 text/html； charset = UTF-8 。

   - pageEncoding：指定当前 JSP 页面的字符编码，通常情况下和contentType中的一致。

   - isELIgnored：指定当前页面是否忽略EL表达式，通常使用false。



## 3 include指令

1. include指令用于通知JSP引擎在翻译当前JSP页面时将其他文件中的内容合并进当前JSP页面转换成的Servlet源文件中，这种在源文件级别进行引入的方式称之为**静态引入**（两个页面合成一个.java文件），当前JSP页面与静态引入的页面紧密结合为**一个Servlet**。
2. 语法：
   	<%@ include file="relativeURL"%>
   	**其中的file属性用于指定被引入文件的相对路径**。  
3. 细节：
   - 被引入的文件必须遵循JSP语法，其中的内容可以包含静态HTML、JSP脚本元素、JSP指令和JSP行为元素等普通JSP页面所具有的一切内容。  
   - 被引入的文件可以使用任意的扩展名，即使其扩展名是html，JSP引擎也会按照处理jsp页面的方式处理它里面的内容，为了见明知意，JSP规范建议使用.jspf（JSP fragments）作为静态引入文件的扩展名。 
   - 在将JSP文件翻译成Servlet源文件时，JSP引擎将合并被引入的文件与当前JSP页面中的指令元素（设置pageEncoding属性的page指令除外），所以，除了import和pageEncoding属性之外，page指令的其他属性不能在这两个页面中有不同的设置值。 