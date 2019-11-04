# JSP（Java Server Pages）概述

## 1 为什么要有JSP

1. 在很多动态网页中，绝大部分内容都是固定不变的，只有局部内容需要动态产生和改变。 
2. 如果使用Servlet程序来输出只有局部内容需要动态改变的网页，其中所有的静态内容也需要程序员用Java程序代码产生，整个Servlet程序的代码将非常臃肿，编写和维护都将非常困难。  
3. 对大量静态内容的美工设计和相关HTML语句的编写，并不是程序员所要做的工作，程序员对此也不一定在行。网页美工设计和制作人员不懂Java编程，更是无法来完成这样的工作。 
4. 为了弥补 Servlet 的缺陷，SUN公司在Servlet的基础上推出了JSP（Java Server Pages）技术作为解决方案。
5. JSP是简化Servlet编写的一种技术，它将Java代码和HTML语句混合在同一个文件中编写，只对网页中的要动态产生的内容采用Java代码来编写，而对固定不变的静态内容采用普通静态HTML页面的方式编写。 



## 2 JSP的hello world

新建一个JSP页面，在body节点内的<% %>中编写java代码

```jsp
<%@page import="java.util.Date"%>
<%@ page language="java" contentType="text/html; charset=ISO-8859-1"
    pageEncoding="ISO-8859-1"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="ISO-8859-1">
<title>Insert title here</title>
</head>
<body>

    <%
        Date date = new Date();
        System.out.println(date);
    %>

</body>
</html>
```



## 3 JSP文件的位置

JSP文件可以放置在WEB应用程序中除了WEB-INF及其子目录外的任何其他目录中，JSP页面的访问路径与普通HTML页面的访问路径形式也完全一样。



## 4 JSP的运行原理

JSP本质上是一个Servlet。

1. WEB容器（Servlet引擎）接收到以.jsp为扩展名的URL的访问请求时，它将把该访问请求交给JSP引擎去处理。
2. 每个JSP 页面在第一次被访问时，JSP引擎将它翻译成一个Servlet源程序，接着再把这个Servlet源程序编译成Servlet的class类文件，然后再由WEB容器（Servlet引擎）像调用普通Servlet程序一样的方式来装载和解释执行这个由JSP页面翻译成的Servlet程序。 
3. JSP规范也没有明确要求JSP中的脚本程序代码必须采用Java语言，JSP中的脚本程序代码可以采用Java语言之外的其他脚本语言来编写，但是，JSP页面最终必须转换成Java Servlet程序。 
4. 可以在WEB应用程序正式发布之前，将其中的所有JSP页面预先编译成Servlet程序。

