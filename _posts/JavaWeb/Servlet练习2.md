# Servlet练习2

## 1 要求

1. 在MySQL数据库中创建一个test_users数据表，添加3个字段，id、user、password，并录入几条记录。
2. 定义一个login.html，定义两个请求字段，user，password。并发送请求到LoginServlet，再创建一个LoginServlet（需要继承自HttpServlet，并重写其doPost方法），在其中获取请求的user，password。
3. 利用JDBC从test_users中查询有没有和页面输入的user，password对应的记录。若有，响应hello:xxx，若没有，响应sorry:xxx



## 2 准备

1. 数据库表

   ![1572854865360](Servlet%E7%BB%83%E4%B9%A02.assets/1572854865360.png)

2. 在web.xml文件中设置Servlet

   ```xml
   <servlet>
       <servlet-name>loginServlet</servlet-name>
       <servlet-class>com.husthuangkai.excercise2.loginservlet.LoginServlet</servlet-class>
     </servlet>
     
     <servlet-mapping>
       <servlet-name>loginServlet</servlet-name>
       <url-pattern>/loginServlet</url-pattern>
     </servlet-mapping>
   ```

3. 编写html页面

   ```html
   <!DOCTYPE html>
   <html>
   <head>
   <meta charset="UTF-8">
   <title>Insert title here</title>
   </head>
   <body>
   
       <form action="loginServlet" method="post">
       
           user: <input type="text" name="user"/>
           password: <input type="password" name="password"/>
           
           <input type="submit" value="Submit"/>
       </form>
   
   </body>
   </html>
   ```

4. 编写LoginServlet类

   ```java
   
   import java.io.IOException;
   import java.io.PrintWriter;
   import java.sql.Connection;
   import java.sql.DriverManager;
   import java.sql.PreparedStatement;
   import java.sql.ResultSet;
   import java.sql.SQLException;
   
   import javax.servlet.ServletException;
   import javax.servlet.http.HttpServlet;
   import javax.servlet.http.HttpServletRequest;
   import javax.servlet.http.HttpServletResponse;
   
   public class LoginServlet extends HttpServlet {
   
       /**
        * 
        */
       private static final long serialVersionUID = 1L;
   
       @Override
       protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {    
           // 获取输入参数
           String inputUserNameString = req.getParameter("user");
           String inputPasswordString = req.getParameter("password");
           
           // 获取数据库连接
           Connection connection = null;
           PreparedStatement statement = null;
           ResultSet resultSet = null;
           
           PrintWriter writer = resp.getWriter();
           
           try {
               // 注册JDBC驱动
               Class.forName("com.mysql.cj.jdbc.Driver");
               
               // 打开链接
               connection = DriverManager.getConnection
                       ("jdbc:mysql://localhost:3306/taotao?useSSL=false&serverTimezone=UTC", "root", "123456");
               
               // 执行查询
               String sqlString = "SELECT COUNT(*) FROM test_users WHERE name = ? AND password = ?";
               statement = connection.prepareStatement(sqlString);
               statement.setString(1, inputUserNameString);
               statement.setString(2, inputPasswordString);
               resultSet = statement.executeQuery();
               
               // 展开查询集
               while (resultSet.next()) {
                   int result = resultSet.getInt(1);
                   if (result > 0) {
                       writer.write("hello: " + inputUserNameString);
                   } else {
                       writer.write("sorry: " + inputUserNameString);
                   }
               }
               
           } catch (Exception e) {
               // TODO: handle exception
               e.printStackTrace();
           } finally {
               if (resultSet != null) {
                   try {
                       resultSet.close();
                   } catch (SQLException e) {
                       // TODO Auto-generated catch block
                       e.printStackTrace();
                   }
               }
               
               if (statement != null) {
                   try {
                       statement.close();
                   } catch (SQLException e) {
                       // TODO Auto-generated catch block
                       e.printStackTrace();
                   }
               }
               
               if (connection != null) {
                   try {
                       connection.close();
                   } catch (SQLException e) {
                       // TODO Auto-generated catch block
                       e.printStackTrace();
                   }
               }
           }
       }
   }
   
   ```



## 3 运行结果

1. 输入 aa, 123

   ![1572855109062](Servlet%E7%BB%83%E4%B9%A02.assets/1572855109062.png)

   输出：

   ![1572855124455](Servlet%E7%BB%83%E4%B9%A02.assets/1572855124455.png)

2. 输入ab，456

   ![1572855196688](Servlet%E7%BB%83%E4%B9%A02.assets/1572855196688.png)

   输出：

   ![1572855207417](Servlet%E7%BB%83%E4%B9%A02.assets/1572855207417.png)

   