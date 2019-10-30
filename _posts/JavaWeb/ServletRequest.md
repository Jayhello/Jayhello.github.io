# ServletRequest

## 1 如何在Servlet中获取请求信息

```java
public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
        // TODO Auto-generated method stub

    }
```

ServletRequest：封装了请求信息，可以从中获取到任何请求信息。

ServletResponse：封装了响应信息，如果想给用户什么响应，具体可以使用该接口的方法来实现。

这两个接口的实现类都是服务器给予实现的，并且在服务器调用service方法时传入。



## 2 ServletRequest：

1. 获取请求参数

   | `String`      | getParameter(String name) Returns the value of a request parameter as a String, or null if the parameter does not exist. |
   | ------------- | ------------------------------------------------------------ |
   | `Map`         | `getParameterMap()` Returns a java.util.Map of the parameters of this request. |
   | `Enumeration` | `getParameterNames()` Returns an `Enumeration` of `String` objects containing the names of the parameters contained in this request. |
   | `String[]`    | `getParameterValues(String name)` Returns an array of `String` objects containing  all of the values the given request parameter has, or  `null` if the parameter does not exist. |

2. 获取请求的URL

   ```java
   	HttpServletRequest httpServletRequest = (HttpServletRequest) servletRequest;
   	String requestURLString = httpServletRequest.getRequestURI();
       System.out.println(requestURLString);
   ```

3. 获取请求的方式

   ```java
       String requestString = httpServletRequest.getMethod();
       System.out.println(requestString);
   ```

   

