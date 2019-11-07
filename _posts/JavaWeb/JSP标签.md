# JSP 标签

## 1 概念

1. JSP还提供了一种称之为Action的元素，在JSP页面中使用Action元素可以完成各种通用的JSP页面功能，也可以实现一些处理复杂业务逻辑的专用功能。 
2. Action元素采用XML元素的语法格式，即每个Action元素在JSP页面中都以XML标签的形式出现。
3. JSP规范中定义了一些标准的Action元素，这些元素的标签名都以jsp作为前缀，并且全部采用小写，例如，\<jsp:include>、\<jsp:forward>等等。  



## 2 \<jsp:include>

1. \<jsp:include>标签用于把另外一个资源的输出内容插入进当前JSP页面的输出内容之中，这种在JSP页面执行时的引入方式称之为**动态引入**。有两个.java源文件。
   语法：
   	<jsp:include page="relativeURL | <%=expression%>" flush="true|false" />
2. page属性用于指定被引入资源的相对路径，它也可以通过执行一个表达式来获得。
   flush属性指定在插入其他资源的输出内容时，是否先将当前JSP页面的已输出的内容刷新到客户端。  
3. \<jsp:include>标签是在当前JSP页面的执行期间插入被引入资源的输出内容，当前JSP页面与被动态引入的资源是两个彼此独立的执行实体，被动态引入的资源必须是一个能独立被WEB容器调用和执行的资源。include指令只能引入遵循JSP格式的文件，被引入文件与当前JSP文件共同合被翻译成一个Servlet的源文件。 



## 3 \<jsp:forward>

相当于请求转发。

但使用\<jsp:forward>标签可以使用\<jsp:param>子标签，向转发页面传入参数。同样\<jsp:include>也可以。





