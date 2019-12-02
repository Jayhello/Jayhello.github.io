# 1 使用Filter作为控制器的MVC

1. Servlet 能做的 Filter 是否都可以完成 ? 这是肯定的。
2. Filter 能做的 Servlet 都可以完成吗 ? 拦截资源却不是 Servlet 所擅长的! Filter 中有一个 FilterChain，这个 API 在 Servlet 中没有！
3. Structs2就是基于这一事实设计的，使用拦截器来完成框架的大部分工作。



# 2 Structs2的Helloworld

1. 搭建环境

   - 导入jar包：复制 struts\apps\struts2-blank\WEB-INF\lib 下的所有 jar 包到当前 web 应用的 lib 目录下。

     ![1575249706774](../images/Structs2%E7%AE%80%E4%BB%8B.assests/1575249706774.png)

     

   - 在web.xml文件中配置structs2：复制 struts\apps\struts2-blank1\WEB-INF\web.xml 文件中的过滤器的配置到当前 web 应用的 web.xml 文件中。

     ```xml
     <!-- 配置 Struts2 的 Filter -->
         <filter>
             <filter-name>struts2</filter-name>
             <filter-class>org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter</filter-class>
         </filter>
     
         <filter-mapping>
             <filter-name>struts2</filter-name>
             <url-pattern>/*</url-pattern>
         </filter-mapping>
     ```

     

   - 在当前web应用的classpath目录下添加structs2的配置文件structs.xml：复制 struts1\apps\struts2-blank\WEB-INF\classes 下的 struts.xml 文件到当前 web 应用的 src 目录下。

2. 配置action

   配置action是关键的一步，假设我们有一个超链接要链接到input.jsp页面，在之前我们要这样写：

   ```html
   <a href="${pageContext.request.contextPath }/helloworld.jsp">转到输入页面2</a>
   ```

   在sturcts中，只需要这样写：

   ```html
   <a href="helloworld.action">转到输入页面1</a>
   ```

   意思很明显，超链接到helloworld.action，那么action是什么呢，这就需要在structs.xml文件中配置。

   ```xml
       <!-- 
           package：包。structs2使用package来组织模块。
           name属性：必须，用于其它的包应用当前包
           extends：当前包继承哪个包，默认情况下继承 structs-default包
        -->
       <package name="helloworld" extends="struts-default">
           <!-- 配置一个action:一个struct2的请求就是要给action
               name：对应一个structs2的请求的名字（或对应一个servletPath，但去除/和扩展名）
            -->
            
            <action name="helloworld">
               <result>/helloworld.jsp</result>
           </action>
       </package>
   ```

   这样就可以转到hellowrold.jsp

   ![1575253726073](../images/Structs2%E7%AE%80%E4%BB%8B.assests/1575253726073.png)

   ![1575253731412](../images/Structs2%E7%AE%80%E4%BB%8B.assests/1575253731412.png)

3. action的完整形式

   在上面一个例子中，action使用了最简化的配法，实际上它的完整形式是这样的：

   ```xml
   <action name="helloworld" class="com.opensymphony.xwork2.ActionSupport" method="execute">
       <result name="success" type="dispatcher">/helloworld.jsp</result>
   </action>
   ```

   

   class：表示这个action的类：默认为com.opensymphony.xwork2.ActionSupport

   method：表示这个action使用类中的方法名，默认为execute方法，返回值类型为String，默认返回值为“success”。

   result：表示根据运行的不同返回值进行不同操作，可以配置多个result，根据返回值的不同进行不同的操作，相当于swtich-case。

   type：指明使用转发还是重定向。

   上面action的完整解释是：

   - 配置一个名为Product-input的action，在访问${contextPath}/Product-input.action时就时访问到这个action。
   - 调用com.opensymphony.xwork2.ActionSupport类的execute方法。
   - 如果方法的返回值为success，则使用转发方式访问/helloworld.jsp。

4. 使用自定义的类来完成action

   - 创建一个Game类，类中有一个game()方法，随意返回"true"或"false"。

     ```java
     package com.husthuangkai.structs2;
     
     import java.util.Random;
     
     public class Game {
         public String game() {
             int random = new Random().nextInt();
             if (random % 2 ==1) {
                 return "true";
             } else {
                 return "false";
             }
         }
     }
     
     ```

     

   - 创建一个action，名为game，使用Game类中的game方法，若返回true，则转发到成功页面，若返回false，则转发到失败页面。

     ```xml
     <action name="game" class="com.husthuangkai.structs2.Game" method="game">
     	<result name="true" type="dispatcher">/WEB-INF/pages/success.jsp</result>
     	<result name="false" type="dispatcher">/WEB-INF/pages/fail.jsp</result>
     </action>
     ```

   - 效果

     ![1575255504837](../images/Structs2%E7%AE%80%E4%BB%8B.assests/1575255504837.png)

     ![1575255510598](../images/Structs2%E7%AE%80%E4%BB%8B.assests/1575255510598.png)

     ![1575255523095](../images/Structs2%E7%AE%80%E4%BB%8B.assests/1575255523095.png)



