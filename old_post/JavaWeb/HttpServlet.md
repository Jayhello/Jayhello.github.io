# HttpServlet

1. 是一个Servlet，继承自GenericServlet，针对于HTTP协议所制定。

2. 在service（）方法中直接把ServletRequest和ServletResponse转为HttpServletRequest和HttpServletResponse

3. 在service（HttpServletRequest，HeepServletResponse）获取了请求方式，并根据请求方式，创建了doXxx（）方法，和getXxx（）方法。

   ```java
   @Override
   public void service(ServletRequest req, ServletResponse res)
           throws ServletException, IOException {
   
           HttpServletRequest  request;
           HttpServletResponse response;
   
           try {
               request = (HttpServletRequest) req;
               response = (HttpServletResponse) res;
           } catch (ClassCastException e) {
               throw new ServletException(lStrings.getString("http.non_http"));
           }
           service(request, response);
       }
   
   protected void service(HttpServletRequest req, HttpServletResponse resp)
           throws ServletException, IOException {
   
           String method = req.getMethod();
   
           if (method.equals(METHOD_GET)) {
               long lastModified = getLastModified(req);
               if (lastModified == -1) {
                   // servlet doesn't support if-modified-since, no reason
                   // to go through further expensive logic
                   doGet(req, resp);
               } else {
                   long ifModifiedSince;
                   try {
                       ifModifiedSince = req.getDateHeader(HEADER_IFMODSINCE);
                   } catch (IllegalArgumentException iae) {
                       // Invalid date header - proceed as if none was set
                       ifModifiedSince = -1;
                   }
                   if (ifModifiedSince < (lastModified / 1000 * 1000)) {
                       // If the servlet mod time is later, call doGet()
                       // Round down to the nearest second for a proper compare
                       // A ifModifiedSince of -1 will always be less
                       maybeSetLastModified(resp, lastModified);
                       doGet(req, resp);
                   } else {
                       resp.setStatus(HttpServletResponse.SC_NOT_MODIFIED);
                   }
               }
   
           } else if (method.equals(METHOD_HEAD)) {
               long lastModified = getLastModified(req);
               maybeSetLastModified(resp, lastModified);
               doHead(req, resp);
   
           } else if (method.equals(METHOD_POST)) {
               doPost(req, resp);
   
           } else if (method.equals(METHOD_PUT)) {
               doPut(req, resp);
   
           } else if (method.equals(METHOD_DELETE)) {
               doDelete(req, resp);
   
           } else if (method.equals(METHOD_OPTIONS)) {
               doOptions(req,resp);
   
           } else if (method.equals(METHOD_TRACE)) {
               doTrace(req,resp);
   
           } else {
               //
               // Note that this means NO servlet supports whatever
               // method was requested, anywhere on this server.
               //
   
               String errMsg = lStrings.getString("http.method_not_implemented");
               Object[] errArgs = new Object[1];
               errArgs[0] = method;
               errMsg = MessageFormat.format(errMsg, errArgs);
   
               resp.sendError(HttpServletResponse.SC_NOT_IMPLEMENTED, errMsg);
           }
       }
   
   protected void doGet(HttpServletRequest req, HttpServletResponse resp)
           throws ServletException, IOException
       {
           String protocol = req.getProtocol();
           String msg = lStrings.getString("http.method_get_not_supported");
           if (protocol.endsWith("1.1")) {
               resp.sendError(HttpServletResponse.SC_METHOD_NOT_ALLOWED, msg);
           } else {
               resp.sendError(HttpServletResponse.SC_BAD_REQUEST, msg);
           }
       }
   
   protected void doPost(HttpServletRequest req, HttpServletResponse resp)
           throws ServletException, IOException {
   
           String protocol = req.getProtocol();
           String msg = lStrings.getString("http.method_post_not_supported");
           if (protocol.endsWith("1.1")) {
               resp.sendError(HttpServletResponse.SC_METHOD_NOT_ALLOWED, msg);
           } else {
               resp.sendError(HttpServletResponse.SC_BAD_REQUEST, msg);
           }
       }
   ```

   

4. 实际开发中，直接集成HttpServlet，并根据请求方式覆写doXxx（）方法接口。

5. 好处：直接针对性的覆盖doXxx（）方法，直接使用HttpServletRequest和HttpServletResponse，无需再进行强转。