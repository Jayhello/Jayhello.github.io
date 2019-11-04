# JSP隐式对象

```java
public void _jspService(HttpServletRequest request,
			HttpServletResponse response)
		throws java.io.IOException, ServletException
{
	JspFactory _jspxFactory = null;
	PageContext pageContext = null;
	HttpSession session = null;
	ServletContext application = null;
	ServletConfig config = null;
	JspWriter out = null;
	Object page = this;
	...
        
    // 使用<% %>编写的代码在此位置，以上声明的对象都可以直接使用。
        
	...
	Throwable exception = 
		org.apache.jasper.runtime.JspRuntimeLibrary.getThrowable(request);
	if (exception != null) {
		response.setStatus(HttpServletResponse.SC_INTERNAL_SERVER_ERROR);
}

```

- request：HttpServletRequest的对象
- response：HttpServletResponse的对象
- pageContext：PageContext的对象，页面的上下文，可以从该对象中获取到其他8个隐含对象，也可以从中获取到当前页面的其他信息。（学习自定义标签时使用它）
- session：代表浏览器和服务器的一次会话，时HttpSession的一个对象，后面详细学习。
- application：代表当前WEB应用，是ServletContext的对象。
- servletConfig：ServletConfig对象，