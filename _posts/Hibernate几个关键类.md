# 1 Hibernate Configuration  

1. Configuration包含的信息

   Configuration类负责管理Hibernate的配置信息。
   包括如下内容：

   - Hibernate运行的底层信息： **数据库的URL、用户名、密码、JDBC驱动类，数据库Dialect,数据库连接池等。**
   - 持久化类与数据表的映射关系（*.hbm.xml 文件）

2. 创建Configuration 的两种方式

   - 属性文件（hibernate.properties）:  

     ```java
     Configuration cfg = new Configuration();
     ```

   - Xml文件（hibernate.cfg.xml）（推荐使用）：

     ```java
     Configuration cfg = new Configuration().configure();
     ```

     或：

     ```java
     File file = new File("hibernte.cfg.xml");
     Configuration cfg = new Configuration().configure(file);
     ```



# 2 Hibernate SessionFactory  

1.  SessionFactory类的特点  

   - 由Configuration通过加载配置文件创建该对象。
   - SessionFactory对象中保存了当前的**数据库配置信息**和所有**映射关系**以及**预定义的SQL语句**。同时，SessionFactory还负责维护Hibernate的**二级缓存**。
   - **预定义SQL语句**，使用Configuration类创建了SessionFactory对象是，已经在SessionFacotry对象中缓存了一些SQL语句，常见的SQL语句是增删改查（通过主键来查询），这样做的目的是效率更高。
   - **一个SessionFactory实例对应一个数据库**，应用从该对象中获得Session实例。
   - SessionFactory是**重量级的**，意味着不能随意创建或销毁它的实例。如果只访问一个数据库，只需要创建一个SessionFactory实例，且在应用初始化的时候完成。
   - SessionFactory需要一个**较大的缓存**，用来存放预定义的SQL语句及实体的映射信息。另外可以配置一个缓存插件，这个插件被称之为Hibernate的二级缓存，被多线程所共享。

2.  使用SessionFactory注意事项  

   一般应用使用一个SessionFactory,最好是应用启动时就完成初始化。

   

   

   

   

   

   

   

   