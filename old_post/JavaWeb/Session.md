# Session

## 1 Session机制

1. session机制采用的是在服务器端保持 HTTP 状态信息的方案 。
2. 服务器使用一种类似于散列表的结构(也可能就是使用散列表)来保存信息。
3. 当程序需要为某个客户端的请求创建一个session时，服务器首先检查这个客户端的请求里是否包含了一个session标识(即sessionId)，如果已经包含一个sessionId则说明以前已经为此客户创建过session，服务器就按照session id把这个session检索出来使用(如果检索不到，可能会新建一个，这种情况可能出现在服务端已经删除了该用户对应的session对象，但用户人为地在请求的URL后面附加上一个JSESSION的参数)。如果客户请求不包含sessionId，则为此客户创建一个session并且生成一个与此session相关联的sessionId，这个session id将在本次响应中返回给客户端保存。



## 2 保存Session id的几种方式

1. 保存session id的方式可以采用cookie，这样在交互过程中浏览器可以自动的按照规则把这个标识发送给服务器。
2. 由于cookie可以被人为的禁用，必须有其它的机制以便在cookie被禁用时仍然能够把session id传递回服务器**，经常采用的一种技术叫做URL重写，就是把session id附加在URL路径的后面，附加的方式也有两种，一种是作为URL路径的附加信息，另一种是作为查询字符串附加在URL后面。**网络在整个交互过程中始终保持状态，就必须在每个客户端可能请求的路径后面都包含这个session id。



## 3 Session cookie

1. session通过SessionID来区分不同的客户, session是以cookie或URL重写为基础的，默认使用cookie来实现，系统会创造一个名为JSESSIONID的输出cookie，这称之为session cookie，以区别persistent cookies(也就是我们通常所说的cookie)，session cookie是存储于浏览器内存中的，并不是写到硬盘上的，通常看不到JSESSIONID，但是当把浏览器的cookie禁止后，web服务器会采用URL重写的方式传递Sessionid，这时地址栏看到。
2. session cookie针对某一次会话而言，会话结束session cookie也就随着消失了，而persistent cookie只是存在于客户端硬盘上的一段文本。 
3. 关闭浏览器，只会是浏览器端内存里的session cookie消失，但不会使保存在服务器端的session对象消失，同样也不会使已经保存到硬盘上的持久化cookie消失。



## 4 Session的创建与删除

1. 一个常见的错误是以为session在有客户端访问时就被创建，然而事实是直到某server端程序(如Servlet)调用HttpServletRequest.getSession(true) 或者 HttpServletRequest.getSession()这样的语句时才会被创建。
2. session在下列情况下被删除：
   A．程序调用HttpSession.invalidate()
   B．距离上一次收到客户端发送的session id时间间隔超过了session的最大有效时间
   C．服务器进程被停止
   **注意：关闭浏览器只会使存储在客户端浏览器内存中的session cookie失效，不会使服务器端的session对象失效。**



## 5 利用URL重写实现Session跟踪 

1. Servlet规范中引入了一种补充的会话管理机制，它允许不支持Cookie的浏览器也可以与WEB服务器保持连续的会话。这种补充机制要求在响应消息的实体内容中必须包含下一次请求的超链接，并将会话标识号作为超链接的URL地址的一个特殊参数。 
2. 将会话标识号以参数形式附加在超链接的URL地址后面的技术称为URL重写。如果在浏览器不支持Cookie或者关闭了Cookie功能的情况下，WEB服务器还要能够与浏览器实现有状态的会话，就必须对所有可能被客户端访问的请求路径（包括超链接、form表单的action属性设置和重定向的URL）进行URL重写。 
3. HttpServletResponse接口中定义了两个用于完成URL重写方法：
   encodeURL方法 
   encodeRedirectURL方法



## 6 利用Session避免表单的重复提交

1. 重复提交的情况
   - 将表单提交到Servlet，Servlet又通过**转发**响应另一个页面，此时地址栏仍然是Servlet的地址，如果此时点击刷新的话，就是**重复提交**。（如果Servlet通过**重定向**来响应新页面，则刷新新页面**没有重复提交**。）
   - 在响应页面未到达时，重复点击提交按钮。
   - 在响应页面点击返回，再点击提交按钮。（注：点击返回后，**刷新表单页面**，再点击提交，就**不算重复提交**。因为刷新后，表单页面已经是一个新的请求页面了。）
2. 如何避免重复提交
   - 在表单中做一个标记，提交到Servlet时，检查标记是否存在。如果标记存在，则说明是第一次提交，移除此标记并执行首次提交的逻辑。如果标记不存在，则说明不是第一次提交，执行重复提交的逻辑。
   - 利用Session来携带此标记（request无法携带标记到一个新的servlet，因为请求到一个新的servlet时，会产生一个新的request）。
     - 在原表单页面中，生成一个随机的 token 值。
     - 将此值分别放入 session 属性中和隐藏域中。
     - 在请求的 servlet 中检查session属性中和隐藏域中的token值是否一致，若一致则说明是首次提交，执行首次提交逻辑并移除session属性中的token值。否则执行重复提交逻辑。