# Servlet 练习1

1. 要求

   实现一个网页，输入用户名和密码，正确值是：

   user：husthuangkai

   password：123456

   如果输入正确，显示

   hello world

   否则，显示

   sorry

2. 实现

   - 在web.xml中添加应用参数

     ```xml
     <context-param>
         <param-name>user</param-name>
         <param-value>husthuangkai</param-value>
       </context-param>
       
       <context-param>
         <param-name>password</param-name>
         <param-value>123456</param-value>
       </context-param>
     ```

   - 写Servlet初始化函数

     ```java
     public void init(ServletConfig arg0) throws ServletException {
             // TODO Auto-generated method stub
             user = arg0.getServletContext().getInitParameter("user");
             password = arg0.getServletContext().getInitParameter("password");
         }
     ```

   - 写service函数

     ```java
      	public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
             // TODO Auto-generated method stub
             
             String userString = servletRequest.getParameter("user");
             String passwordString = servletRequest.getParameter("password");
             
             if (userString.equals(user) && passwordString.equals(password)) {
                 servletResponse.getWriter().write("hello world");
             } else {
                 servletResponse.getWriter().write("sorry");
             }
      	}
     ```

3. 结果

   ![1572600316871](Servlet%E7%BB%83%E4%B9%A01.assets/1572600316871.png)

   - 输入正确用户名和密码

     ![1572600342970](Servlet%E7%BB%83%E4%B9%A01.assets/1572600342970.png)

   - 输入错误用户名和密码

     ![1572600362242](Servlet%E7%BB%83%E4%B9%A01.assets/1572600362242.png)

     