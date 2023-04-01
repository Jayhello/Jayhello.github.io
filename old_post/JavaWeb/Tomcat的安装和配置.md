# Tomcat的安装和配置

## 1 JavaWeb应用的概念

- 在Sun的Java Servlet规范中，对Java Web应用作了这样定义：“Java Web应用由一组Servlet、HTML页、类、以及其它可以被绑定的资源构成。它可以在各种供应商提供的实现Servlet规范的 Servlet容器 中运行。”（Servlet本质上就是运行在服务器上的类）
- Java Web应用中可以包含如下内容:
  - Servlet
  - JSP
  - 实用类
  - 静态文档如HTML、图片等
  - 描述Web应用的信息（web.xml）

## 2 Servlet容器的概念

![1572054994959](Tomcat%E7%9A%84%E5%AE%89%E8%A3%85%E5%92%8C%E9%85%8D%E7%BD%AE.assets/1572054994959.png)

- Servlet容器为JavaWeb应用提供运行时环境，它负责管理Servlet和JSP的生命周期，以及管理它们的共享数据。
- Servlet容器也称为JavaWeb应用容器，或者Servlet/JSP容器。
- 目前最流行的Servlet容器软件括:
  - Tomcat
  - Resin
  - J2EE服务器（如Weblogic）中也提供了内置的Servlet容器

## 3 Web程序结构

![image-20191026102131431](Tomcat%E7%9A%84%E5%AE%89%E8%A3%85%E5%92%8C%E9%85%8D%E7%BD%AE.assets/image-20191026102131431.png)

