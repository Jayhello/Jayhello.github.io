# GenericServlet

1. 是一个Servlet，是Servlet接口和ServletConfig接口的实现类。但是是一个抽象类，其中的service方法为抽象方法。
2. 如果新建的Servlet程序直接继承GenericServlet会使开发更简介。
3. 具体实现：
   - 在GenericServlet中声明了一个ServletConfig类型的成员变量，在init（ServletConfig）方法中对其进行了初始化。
   - 利用servletConfig成员变量的方法实现了ServletConfig接口的方法
   - 还定义了一个init（）方法，在init（ServletConfig）方法中对其调用，子类可以直接覆盖init（）方法，在其中对servlet进行初始化。
   - 不建议直接覆盖init（ServletConfig），因为如果忘记编写super.init(ServletConfig)，并且使用了ServletConfig接口的方法，则会出现空指针异常。
   - 新建的init（）并非Servlet的生命周期方法，而init（ServletConfig）是生命周期相关的方法。