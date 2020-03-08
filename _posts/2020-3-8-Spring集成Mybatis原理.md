---
title: Spring集成Mybatis原理
layout: post
categories: spring
tags: spring
---
* content
{:toc}


ssm框架是目前使用频率比较高的框架，而spring提供的很好的扩展性，让我们在使用spring整合mybatis的时候可以方便的使用mybatis，仅需要少量的配置或直接通过注解就能够将mybatis交给spring容器管理。实际上，最少只需要一个简单的注解就可以做到，那么spring和mybatis究竟是如何做到这一点的呢？了解了这些对于我们自己扩展spring框架又有什么帮助呢？




### 1 spring集成mybatis

首先实现一个spring集成mybatis：

1. 添加依赖

   ```xml
   <dependencies>
           <!-- https://mvnrepository.com/artifact/org.mybatis/mybatis -->
           <dependency>
               <groupId>org.mybatis</groupId>
               <artifactId>mybatis</artifactId>
               <version>3.5.4</version>
           </dependency>
   
           <!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
           <dependency>
               <groupId>mysql</groupId>
               <artifactId>mysql-connector-java</artifactId>
               <version>8.0.17</version>
           </dependency>
   
           <!-- https://mvnrepository.com/artifact/org.mybatis/mybatis-spring -->
           <dependency>
               <groupId>org.mybatis</groupId>
               <artifactId>mybatis-spring</artifactId>
               <version>2.0.3</version>
           </dependency>
   
           <!-- https://mvnrepository.com/artifact/org.springframework/spring-context -->
           <dependency>
               <groupId>org.springframework</groupId>
               <artifactId>spring-context</artifactId>
               <version>5.2.4.RELEASE</version>
           </dependency>
   
           <!-- https://mvnrepository.com/artifact/org.springframework/spring-jdbc -->
           <dependency>
               <groupId>org.springframework</groupId>
               <artifactId>spring-jdbc</artifactId>
               <version>5.2.4.RELEASE</version>
           </dependency>
       </dependencies>
   ```

2. 写配置类

   ```java
   // 扫描所有bean
   @ComponentScan("com")
   // 扫描所有Mybatis的Mapper
   @MapperScan("com.husthuangkai.dao")
   // 配置类
   @Configuration
   public class Config {
       
       // 将SqlSessionFactoryBean加入容器
       @Bean
       public SqlSessionFactoryBean sqlSessionFactoryBean(DataSource dataSource) {
           SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
           sqlSessionFactoryBean.setDataSource(dataSource());
           return sqlSessionFactoryBean;
       }
       
       // 将数据源加入容器
       @Bean
       public DataSource dataSource() {
           DriverManagerDataSource dataSource = new DriverManagerDataSource();
           dataSource.setUrl("jdbc:mysql://localhost:3306/test?useSSL=false&serverTimezone=UTC");
           dataSource.setPassword("root");
           dataSource.setUsername("root");;
           dataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
           return dataSource;
       }
   }
   ```

3. POJO

   ```java
   public class Customer {
   
       private Integer id;
       private String name;
       private String address;
       private String phone;
       //...get
       //...set
   }
   ```

4. Mapper

   ```java
   public interface CustomerMapper {
       // 通过注解的方式配置语句
       @Select("SELECT * from customer where id = #{id}")
       Customer selectCustomer(int id);
   }
   ```

5. service

   ```java
   @Service
   public class CustomerService {
       @Autowired
       CustomerMapper customerMapper;
       
       public Customer getCustomerById(int id) {
           return customerMapper.selectCustomer(id);
       }
   }
   ```

6. main

   ```java
   public class Main {
   
       
       public static void main(String[] args) throws IOException {
           // 根据配置类创建IOC容器
           ApplicationContext applicationContext = new AnnotationConfigApplicationContext(Config.class);
           // 从容器中获取servicebean
           CustomerService service = (CustomerService) applicationContext.getBean("customerService");
           // 使用service
           System.out.println(service.getCustomerById(20));
       }
   }
   
   ```



### 2 关键问题

通过上面的流程就可以将Spring和Mybatis整合起来，其中关键的一步在于，Spring是如何将CustomerMapper进行实例化的，因为我们只写了一个CustomerMapper的接口，并没有实现类。我们要解决的主要是两个问题：

1. CustomerMapper是如何实例化的。
2. CustomerMapper实例化后是如何加入IOC容器的。

```java
public interface CustomerMapper {
    // 通过注解的方式配置语句
    @Select("SELECT * from customer where id = #{id}")
    Customer selectCustomer(int id);
}

@Service
public class CustomerService {
    // 这个是如何实例化并注入的
    @Autowired
    CustomerMapper customerMapper;
    
    // ...
}
```



### 3 Mapper实例化

那么猜测一下，这个CustomerMapper的实例化对象，可能是通过动态代理产生的。那么来进行一个证明。一个简单的证明方法就是进行调试：

1. 通过调试证明

   ![image-20200308161706243](../images/Spring%E9%9B%86%E6%88%90Mybatis%E5%8E%9F%E7%90%86.assets/image-20200308161706243.png)

   ![image-20200308161846207](../images/Spring%E9%9B%86%E6%88%90Mybatis%E5%8E%9F%E7%90%86.assets/image-20200308161846207.png)

   我们可以看到service中的customerMapper对象是一个$Proxy19的类型，说明了它是一个动态代理产生的对象。

2. 通过源码证明

   这里需要注意一个问题，这一块的原理实际上和spring没有半毛钱关系。我们回忆一下，我们单独使用mybatis的时候，大致流程是这样的：

   - 根据配置文件创建一个sqlSessionFactory
   - 从sqlSessionFactory中获取一个session
   - 调用session.getMapper()方法获得mapper
   - 使用mapper

   在单独使用mybatis的时候我们的Mapper也是个接口，看如下的代码可能更清晰一点：

   ```java
   // 配置文件的路径
   String resource = "org/mybatis/example/mybatis-config.xml";
   InputStream inputStream = Resources.getResourceAsStream(resource);
   // 获取sqlSessionFactory
   SqlSessionFactory sqlSessionFactory =
     new SqlSessionFactoryBuilder().build(inputStream);
   // 获取session
   try (SqlSession session = sqlSessionFactory.openSession()) {
       // 通过动态代理获取BolgMapper接口的实例化对象
     BlogMapper mapper = session.getMapper(BlogMapper.class);
       // 使用bolgMapper
     Blog blog = mapper.selectBlog(101);
   }
   ```
   那么我们就去看一下这个session.getMapper()。它实际上调用的是DefaultSqlSession类中的getMapper()。

   ```java
     @Override
     public <T> T getMapper(Class<T> type) {
       return configuration.getMapper(type, this);
     }
   
     public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
       return mapperRegistry.getMapper(type, sqlSession);
     }
   
     public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
         // 这是一个mapper代理工厂
       final MapperProxyFactory<T> mapperProxyFactory = (MapperProxyFactory<T>) knownMappers.get(type);
       if (mapperProxyFactory == null) {
         throw new BindingException("Type " + type + " is not known to the MapperRegistry.");
       }
       try {
           // 通过mapper代理工厂代理出Mapper的实例
         return mapperProxyFactory.newInstance(sqlSession);
       } catch (Exception e) {
         throw new BindingException("Error getting mapper instance. Cause: " + e, e);
       }
     }
   
		// 下面就是典型的动态代理了
     public T newInstance(SqlSession sqlSession) {
       final MapperProxy<T> mapperProxy = new MapperProxy<>(sqlSession, mapperInterface, methodCache);
       return newInstance(mapperProxy);
     }
   
     protected T newInstance(MapperProxy<T> mapperProxy) {
       return (T) Proxy.newProxyInstance(mapperInterface.getClassLoader(), new Class[] { mapperInterface }, mapperProxy);
     }
   ```

   通过以上的源码，我们证明了mybatis确实是通过动态代理来实例化Mapper接口的对象的。



### 4 Mapper实例化对象加入IOC容器

我们来找一下我们的代码，还有哪些东西可能会把这个Mapper实例化加入IOC容器呢？只有可能是这个注解了：

```java
// 扫描所有Mybatis的Mapper
@MapperScan("com.husthuangkai.dao")
```

因为它扫描了所有的Mapper，所以它的"嫌疑最大“，并且，也没有其他地方有可能了。我们点开看一下：

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Import(MapperScannerRegistrar.class)
@Repeatable(MapperScans.class)
public @interface MapperScan {
    //...
}
```

那么它上面的这五个注解都有什么用呢？经过调查，”罪魁祸首“正是这个@Import注解。在这五个注解中，只有这个@Import注解是来自spring的，所以，就是它使mybatis与spring产生了关联。

```java
@Import(MapperScannerRegistrar.class)
```

这个@Import可以将一个类注入到IOC容器中。而这个MapperScannerRegistrar.class是什么呢？

```java
public class MapperScannerRegistrar implements ImportBeanDefinitionRegistrar, ResourceLoaderAware {
    // ...
}
```

这个类实现了ImportBeanDefinitionRegistrar接口，通过查springframework的文档得到这个接口的作用：

> ```
> public interface ImportBeanDefinitionRegistrar
> ```
>
> Interface to be implemented by types that register additional bean definitions when processing @[`Configuration`](https://docs.spring.io/spring/docs/5.2.4.RELEASE/javadoc-api/org/springframework/context/annotation/Configuration.html) classes. Useful when operating at the bean definition level (as opposed to `@Bean` method/instance level) is desired or necessary.
>
> Along with `@Configuration` and [`ImportSelector`](https://docs.spring.io/spring/docs/5.2.4.RELEASE/javadoc-api/org/springframework/context/annotation/ImportSelector.html), classes of this type may be provided to the @[`Import`](https://docs.spring.io/spring/docs/5.2.4.RELEASE/javadoc-api/org/springframework/context/annotation/Import.html) annotation (or may also be returned from an `ImportSelector`).
>
> An [`ImportBeanDefinitionRegistrar`](https://docs.spring.io/spring/docs/5.2.4.RELEASE/javadoc-api/org/springframework/context/annotation/ImportBeanDefinitionRegistrar.html) may implement any of the following [`Aware`](https://docs.spring.io/spring/docs/5.2.4.RELEASE/javadoc-api/org/springframework/beans/factory/Aware.html) interfaces, and their respective methods will be called prior to [`registerBeanDefinitions(org.springframework.core.type.AnnotationMetadata, org.springframework.beans.factory.support.BeanDefinitionRegistry, org.springframework.beans.factory.support.BeanNameGenerator)`](https://docs.spring.io/spring/docs/5.2.4.RELEASE/javadoc-api/org/springframework/context/annotation/ImportBeanDefinitionRegistrar.html#registerBeanDefinitions-org.springframework.core.type.AnnotationMetadata-org.springframework.beans.factory.support.BeanDefinitionRegistry-org.springframework.beans.factory.support.BeanNameGenerator-):
>
> - [`EnvironmentAware`](https://docs.spring.io/spring/docs/5.2.4.RELEASE/javadoc-api/org/springframework/context/EnvironmentAware.html)
> - [`BeanFactoryAware`](https://docs.spring.io/spring/docs/5.2.4.RELEASE/javadoc-api/org/springframework/beans/factory/BeanFactoryAware.html)
> - [`BeanClassLoaderAware`](https://docs.spring.io/spring/docs/5.2.4.RELEASE/javadoc-api/org/springframework/beans/factory/BeanClassLoaderAware.html)
> - [`ResourceLoaderAware`](https://docs.spring.io/spring/docs/5.2.4.RELEASE/javadoc-api/org/springframework/context/ResourceLoaderAware.html)
>
> Alternatively, the class may provide a single constructor with one or more of the following supported parameter types:
>
> - [`Environment`](https://docs.spring.io/spring/docs/5.2.4.RELEASE/javadoc-api/org/springframework/core/env/Environment.html)
> - [`BeanFactory`](https://docs.spring.io/spring/docs/5.2.4.RELEASE/javadoc-api/org/springframework/beans/factory/BeanFactory.html)
> - [`ClassLoader`](https://docs.oracle.com/javase/8/docs/api/java/lang/ClassLoader.html?is-external=true)
> - [`ResourceLoader`](https://docs.spring.io/spring/docs/5.2.4.RELEASE/javadoc-api/org/springframework/core/io/ResourceLoader.html)
>
> See implementations and associated unit tests for usage examples.

大意就是：

- 当编写配置类的时候，可以通过继承这个接口来注册新增的bean definitions。
- 可以使用@Import注解来导入这个接口的实现类。
- 实现了这个接口的类的方法将会被调用，用来注册bean definitions。

如果看过一点spring源码的同学会知道，BeanDefinition就是用来定义bean的类，spring在初始化容器的时候会先将配置都扫描成BeanDefinition，再通过BeanDefinition来生成bean，因此，只要向容器中注入了BeanDefinition，那么就可以从容器中获取到响应的bean。我们来看一下具体的源码。

```java
public class MapperScannerRegistrar implements ImportBeanDefinitionRegistrar, ResourceLoaderAware {
  @Override
  public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
      // 获得这个注解所在的类上的所有注解的参数，写到一个个hashmap中
      // 比如我们的@MapperScan("com.husthuangkai.dao")中的
      // "com.husthuangkai.dao"就存在这里面
    AnnotationAttributes mapperScanAttrs = AnnotationAttributes.fromMap(importingClassMetadata.getAnnotationAttributes(MapperScan.class.getName()));
    if (mapperScanAttrs != null) {
        // 然后跳转到另一个同名函数
      registerBeanDefinitions(mapperScanAttrs, registry, generateBaseBeanName(importingClassMetadata, 0));
    }
  }

  void registerBeanDefinitions(AnnotationAttributes annoAttrs, BeanDefinitionRegistry registry, String beanName) {
	// 跳转到这里
    // 这个顾名思义，就是用来生成BeanDefinition的builder 
    BeanDefinitionBuilder builder = BeanDefinitionBuilder.genericBeanDefinition(MapperScannerConfigurer.class);
      
    // 这里一大串都是给buider加一些属性，对我们的Mapper没影响，所以略掉了
      // ...
    
    // 这里是将要扫描的报名都放入一个list，"com.husthuangkai.dao"就放在这里
    List<String> basePackages = new ArrayList<>();
      basePackages.addAll(Arrays.stream(annoAttrs.getStringArray("value")).filter(StringUtils::hasText).collect(Collectors.toList()));

    basePackages.addAll(Arrays.stream(annoAttrs.getStringArray("basePackages")).filter(StringUtils::hasText)
        .collect(Collectors.toList()));

    basePackages.addAll(Arrays.stream(annoAttrs.getClassArray("basePackageClasses")).map(ClassUtils::getPackageName)
        .collect(Collectors.toList()));

    // 获得是否懒加载属性，对我们来说不重要
    String lazyInitialization = annoAttrs.getString("lazyInitialization");
    if (StringUtils.hasText(lazyInitialization)) {
      builder.addPropertyValue("lazyInitialization", lazyInitialization);
    }

    // 此处将包名放入了builder
    builder.addPropertyValue("basePackage", StringUtils.collectionToCommaDelimitedString(basePackages));

    // 使用builder注册BeanDefinition
    registry.registerBeanDefinition(beanName, builder.getBeanDefinition());
  }
}

```
到这里我们已经基本弄清了mybatis和spring集成的原理了。

- 首先，在Spring启动的时候，会根据MapperScan提供的包名，将包中的接口都封装为BeanDefination（需要注意的是，这个BeanDefination还不是直接定义Mapper类的，而是定义了一个FactoryBean，这个FactoryBean可以通过动态代理生成一个Mapper类的实例化对象）。
- 当需要Mapper接口的实例化对象的时候，会调用到spring容器的doGetBean()，而doGetBean()会调用FactoryBean的getObject()方法，而这个getObject()方法就会使用动态代理来真正实例化Mapper的对象。

### 4 总结

通过对Spring整合Mybatis的学习，现在我们知道如果我们要实现将自己的框架交给容器管理，并且不需要手动进行很多额外的工作的话，那么我们只需要按照以下步骤来做：

1. 自定义一个类，实现ImportBeanDefinitionRegistrar接口。
2. 实现此接口的registerBeanDefinitions方法，将需要交给IOC容器管理的类信息封装在BeanDefinitionBuilder中，注册到Spring中。
3. 使用@Import将此类导入SpringIOC容器。