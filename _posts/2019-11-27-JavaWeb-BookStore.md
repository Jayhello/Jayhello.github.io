---
title: JavaWeb案例BookStore(1)
layout: post
categories: Java
tags: java基础
---
* content
{:toc}
这个是学习JavaWeb最后地一个练习案例，实现一个简单的网上书城。





# 1 配置环境

1. 配置c3p0数据源

   - 先加入两个jar包：

     c3p0-0.9.1.2.jar

     mysql-connector-java-8.0.16.jar

   - 编辑配置文件

     在src中加入c3p0-config.xml，内容如下：

     ```xml
     <?xml version="1.0" encoding="UTF-8"?>
     <c3p0-config>
       <named-config name="mvc"> 
       
         <property name="user">root</property>
         <property name="password">123456</property>
         <property name="driverClass">com.mysql.cj.jdbc.Driver</property>
         <property name="jdbcUrl">jdbc:mysql://localhost:3306/test?useSSL=false&amp;serverTimezone=UTC&amp;allowPublicKeyRetrieval=true</property>
       
         <property name="acquireIncrement">5</property>
         <property name="initialPoolSize">10</property>
         <property name="minPoolSize">5</property>
         <property name="maxPoolSize">50</property>
     
         <property name="maxStatements">20</property> 
         <property name="maxStatementsPerConnection">5</property>
     
       </named-config>
     </c3p0-config>
     ```

2. 