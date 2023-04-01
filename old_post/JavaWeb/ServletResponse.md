# ServletResponse

## 1 简单用法

1. 设置响应的内容

   ```java
   		PrintWriter printWriter = servletResponse.getWriter();
           printWriter.write("hello world");
   ```

2. 设置响应类型

   ```java
   		servletResponse.setContentType("application/msword");
   ```

   

