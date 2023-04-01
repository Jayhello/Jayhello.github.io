# 1 在Action中访问WEB资源

1. 什么时WEB资源？

   HttpServletRequest、HttpSession、ServletContext等原生的ServletAPI。

2. 为什么需要访问WEB资源？

   B\S的应用的Controller中必然需要访问WEB资源。

3. 如何访问？

   - 和ServletAPI解耦的方式：只能访问有限的ServletAPI对象，且只能访问其有限的方法（读取请求参数，读写域对象的属性等）

     - 使用ActionContext

       在jsp中为application配置一个属性。

       添加到test.action的超链接，设置请求参数为name=123&name=456。

       ```jsp
       <%application.setAttribute("applicationAttribute", "applicationAttributeValue"); %>
       <a href="test.action?name=123&name=456">test</a>
       ```

       配置action

       ```xml
       <action name="test" class="com.husthuangkai.structs2.action.TestAction" method="test">
       	<result name="success" type="dispatcher">/success.jsp</result>
       </action>
       ```

       写TestAction类

       ```java
       package com.husthuangkai.structs2.action;
       
       import java.util.Map;
       
       import com.opensymphony.xwork2.ActionContext;
       
       public class TestAction {
       
           public String test() {
               
               // 1. 获取application的属性map
               Map<String, Object> applicationMap = ActionContext.getContext().getApplication();
               System.out.println(applicationMap.get("applicationAttribute"));
               applicationMap.put("applicationKey", "applicationValue");
               
               // 2. 获取session的属性map
               Map<String, Object> sessionMap = ActionContext.getContext().getSession();
               sessionMap.put("sessionKey", "sessionValue");
               
               // 3. 获取request的属性map
               Map<String, Object> requestMap = (Map<String, Object>) ActionContext.getContext().get("request");
               requestMap.put("requestKey", "requestValue");
               
               // 4. 获取请求参数的map
               // 注意：这个Map中的Value是String[]类型，表示该请求参数的name对应的value数组
               // 且这个Map只能读，不能写，即使写了，在下个页面中也不会生效
               Map<String, Object> paramMap = ActionContext.getContext().getParameters();
               String[] values = (String[]) paramMap.get("name");
               for(String value : values) {
                   System.out.println(value);
               }
                
               return "success";
           }
           
       }
       
       ```

       写success.jsp

       ```jsp
       ${applicationScope.applicationKey }
       <br>
       ${sessionScope.sessionKey }
       <br>
       ${requestScope.requestKey }
       <br>
       
       <a href="${pageContext.request.contextPath }/index.jsp">返回</a>
       ```

       效果：

       ![1575262373046](../images/Struct2-Action.assests/1575262373046.png)

       ![1575262379922](../images/Struct2-Action.assests/1575262379922.png)

       ![1575262384474](../images/Struct2-Action.assests/1575262384474.png)

       

     - 实现XxxAware接口

       jsp不变。

       配置action：

       ```xml
       <action name="testAware" class="com.husthuangkai.structs2.action.TestAware" method="testAware">
       	<result name="success" type="dispatcher">/success.jsp</result>
       </action>
       ```

       写TestAware类：

       ```java
       package com.husthuangkai.structs2.action;
       
       import java.util.Map;
       
       import org.apache.struts2.interceptor.ApplicationAware;
       import org.apache.struts2.interceptor.ParameterAware;
       import org.apache.struts2.interceptor.RequestAware;
       import org.apache.struts2.interceptor.SessionAware;
       
       public class TestAware implements ApplicationAware, SessionAware, RequestAware, ParameterAware {
       
           private Map<String, Object> applicationMap = null;
           private Map<String, Object> sessionMap = null;
           private Map<String, Object> requestMap = null;
           private Map<String, String[]> paramMap = null;
           
           public String testAware() {
               applicationMap.put("applicationKey", "applicationValue");
               sessionMap.put("sessionKey", "sessionValue");
               requestMap.put("requestKey", "requestValue");
               for (String param : paramMap.get("name")) {
                   System.out.println(param);
               }
               return "success";
           }
           
           @Override
           public void setApplication(Map<String, Object> arg0) {
              applicationMap = arg0;
           }
       
           @Override
           public void setRequest(Map<String, Object> arg0) {
               requestMap = arg0;
           }
       
           @Override
           public void setSession(Map<String, Object> arg0) {
               sessionMap = arg0;
           }
       
           @Override
           public void setParameters(Map<String, String[]> arg0) {
               paramMap = arg0;
           }
       }
       
       ```

     - 选用的建议：若一个Action类中有多个action方法，且多个方法都需要使用域对象的Map或parameters，则建议使用Aware接口方式。

     - session对应的Map实际上是SessionMap类型的，强转后可以调用其invalidate方法使session失效。

   - 和ServletAPI耦合的方式：可以访问更多的ServletAPI对象，且可以调用其原生方法。

     - 使用ServletActionContext

       ```java
       package com.husthuangkai.structs2.action;
       
       import javax.servlet.ServletContext;
       import javax.servlet.http.HttpServletRequest;
       import javax.servlet.http.HttpSession;
       
       import org.apache.struts2.ServletActionContext;
       
       public class TestServletActionContext {
       
           public String testServletActionContext() {
               HttpServletRequest request = ServletActionContext.getRequest();
               HttpSession session = request.getSession();
               ServletContext context = ServletActionContext.getServletContext();
               
               return "success";
           }
           
       }
       
       ```

       

     - 实现ServletXxxAware接口

       ```java
       package com.husthuangkai.structs2.action;
       
       import javax.servlet.ServletContext;
       import javax.servlet.http.HttpServletRequest;
       import javax.servlet.http.HttpServletResponse;
       import javax.servlet.http.HttpSession;
       
       import org.apache.struts2.interceptor.ServletRequestAware;
       import org.apache.struts2.interceptor.ServletResponseAware;
       import org.apache.struts2.util.ServletContextAware;
       
       
       public class TestServletAware implements ServletRequestAware, ServletContextAware, ServletResponseAware {
       
           private HttpServletRequest request = null;
           private HttpSession session = null;
           private HttpServletResponse response = null;
           private ServletContext context = null;
           
           @Override
           public void setServletResponse(HttpServletResponse arg0) {
               response = arg0;
           }
       
           @Override
           public void setServletContext(ServletContext arg0) {
               context = arg0;
           }
       
           @Override
           public void setServletRequest(HttpServletRequest arg0) {
               request = arg0;
               session = request.getSession();
           }
       
       }
       
       ```



# 2 关于Action类

1. 将Action类的字段名称按照JavaBeans的命名规则来命名，并生成getter和setter方法，这样如果有同名的请求参数，structs框架就会自动将请求参数赋给这个字段。名且会自动进行字符串到其他类型的转换。
2. 必须有无参构造器，因为action方法需要通过反射来调用。
3. 至少有一个供action调用的方法。
4. structs会为每一个请求创建一个新的Action实例，即Action不是单例的。



# 3 关于struts请求的扩展名问题

默认情况下，struts可以拦截扩展名为.action和没有扩展名的请求，这配置在struts2-core-2.3.15.3.jar/org/apache/struts2/default.properties文件中。

> struts.action.extension=action,,

可以在structs的配置文件中对这个值进行配置

```xml
<constant name="struts.action.extension" value="do,action"></constant>
```



# 4 result

1. result是action则子节点，根据method方法的返回值来决定去往哪个result。
2. 一个action可以有多个result子节点。
3. result有两个属性，name表示method可能的返回值。
4. type表示处理结果的方式，常用的有以下几种类型：
   - **dispatcher**转发（不可以转发到action）
   - **redirect**重定向（也可以重定向到action）
   - **redirectAction**重定向到action
   - **chain**转发到action



# 5 Action的通配符映射

1. 一个 Web 应用可能有成百上千个 action 声明. 可以利用 struts 提供的通配符映射机制把多个彼此相似的映射关系简化为一个映射关系。
2. 通配符映射规则
   - 若找到多个匹配, 没有通配符的那个将胜出
   - 若指定的动作不存在, Struts 将会尝试把这个 URI 与任何一个包含着通配符 * 的动作名及进行匹配	
   - 被通配符匹配到的 URI 字符串的子串可以用 {1}, {2} 来引用. {1} 匹配第一个子串, {2} 匹配第二个子串…
   - {0} 匹配整个 URI
   - 若 Struts 找到的带有通配符的匹配不止一个, 则按先后顺序进行匹配
   - *可以匹配零个或多个字符, 但不包括 / 字符. 如果想把 / 字符包括在内, 需要使用 **。如果需要对某个字符进行转义, 需要使用 \\。



# 6 动态方法调用

通常，请求一个action时会调用action中配置的method属性对应的方法，动态方法就用就是可以通过以下的形式来指定调用哪个方法。

```html
<!-- methodName就是要动态调用 -->
<a href="test!methodName.do">DynamicMethod</a>
```

默认情况下这个功能是关闭的，需要在struts.xml配置文件中加入如下静态属性启用。

```html
<constant name="struts.enable.DynamicMethodInvocation" value="true"></constant>
```

通常情况下不这样使用，因为这样会在url中暴露内部实现，并且不方便管理。

