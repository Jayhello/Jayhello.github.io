# JSP基本语法

## 1 模板元素

1. JSP页面中的静态HTML内容称之为JSP模版元素，在静态的HTML内容之中可以嵌套JSP的其他各种元素来产生动态内容和执行业务逻辑。 
2. JSP模版元素定义了网页的基本骨架，即定义了页面的结构和外观。



## 2 JSP表达式

JSP表达式（expression）提供了将一个java变量或表达式的计算结果输出到客户端的简化方式，它将要输出的变量或表达式直接封装在<%= 和 %>之中。

```jsp
	<%
        Date date = new Date();
    %>
    <%=date%>
```



## 3 JSP脚本片段

```jsp
	<%
        Date date = new Date();
        System.out.println(date);
    %>
```



## 4 JSP声明

JSP声明将Java代码封装在<%！和 %>之中，它里面的代码将被插入进Servlet的_jspService方法的外面，所以，JSP声明可用于定义JSP页面转换成的Servlet程序的静态代码块、成员变量和方法 。 （在JSP页面中几乎从不这样使用）。



## 5 注释

```jsp
<!--HTML注释-->
<%--JSP注释--%>
```

JSP注释可以阻止Java代码的执行。