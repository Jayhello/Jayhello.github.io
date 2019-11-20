---
title: Spring 中的事务管理
layout: post
categories: Java
tags: spring
---
* content
{:toc}






# Spring 中的事务管理

## 1 事务简介

事务管理是企业级应用程序开发中必不可少的技术,  用来确保数据的完整性和一致性. 
事务就是一系列的动作, 它们被当做一个单独的工作单元. 这些动作要么全部完成, 要么全部不起作用
事务的四个关键属性(ACID)

- 原子性(atomicity): 事务是一个原子操作, 由一系列动作组成. 事务的原子性确保动作要么全部完成要么完全不起作用。
- 一致性(consistency): 一旦所有事务动作完成, 事务就被提交. 数据和资源就处于一种满足业务规则的一致性状态中
- 隔离性(isolation): 可能有许多事务会同时处理相同的数据, 因此每个事物都应该与其他事务隔离开来, 防止数据损坏
- 持久性(durability): 一旦事务完成, 无论发生什么系统错误, 它的结果都不应该受到影响. 通常情况下, 事务的结果被写到持久化存储器中.

## 2 事务管理的问题

- 必须为不同的方法重写类似的样板代码
- 这段代码是特定于 JDBC 的, 一旦选择类其它数据库存取技术, 代码需要作出相应的修改

![](Spring%20%E4%B8%AD%E7%9A%84%E4%BA%8B%E5%8A%A1%E7%AE%A1%E7%90%86.assets/1571735884261-1571811056946.png)



## 3 Spring中的事务管理

- 作为企业级应用程序框架, Spring 在不同的事务管理 API 之上定义了一个抽象层. 而应用程序开发人员不必了解底层的事务管理 API, 就可以使用 Spring 的事务管理机制。
- Spring 既支持编程式事务管理, 也支持声明式的事务管理。
- 编程式事务管理: 将事务管理代码嵌入到业务方法中来控制事务的提交和回滚. 在编程式管理事务时, 必须在每个事务操作中包含额外的事务管理代码。
- **声明式事务管理**: 大多数情况下比编程式事务管理更好用. 它将事务管理代码从业务方法中分离出来, 以声明的方式来实现事务管理. 事务管理作为一种横切关注点, 可以通过 AOP 方法模块化. Spring 通过 Spring AOP 框架支持声明式事务管理.

## 4 举例

### 4.1 数据库表

书

| isbn | book_name | price |
| ---- | --------- | ----- |
| 1    | Java      | 50    |
| 2    | C++       | 55    |



库存

| isbn | stock |
| ---- | ----- |
| 1    | 10    |
| 2    | 10    |



用户账户

| id   | user_name | balance |
| ---- | --------- | ------- |
| 1    | hk        | 200     |
| 2    | ff        | 300     |

配置文件

```xml
<!-- 配置事务管理器 -->
	<bean id="dataSourceTransactionManager" 
		class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		<property name="dataSource" ref="dataSource"></property>
	</bean>
	
	<!-- 启用事务注解 -->
	<tx:annotation-driven transaction-manager="dataSourceTransactionManager"/>
```





服务

```java
public class BookStoreServiceImpl implements BookShopService {

	@Autowired
	BookShopDao bookShopDao;
	
	@Transactional
	@Override
    // 用户买书事务,如果事务途中出现异常，则会回滚
	public void purchase(int userid, String isbn) {
		// TODO Auto-generated method stub
		// 更新库存
		bookShopDao.updateBookStock(isbn);
		
        // 查询价格
		double price = bookShopDao.findBookPriceByIsbn(isbn);
		
        // 更新余额
		bookShopDao.updateUserBalance(userid, price);
	}

}
```



```java
@Repository
public class BookShopDaoImpl implements BookShopDao {

	@Autowired
	private JdbcTemplate jdbcTemplate;

	// 查价格
	@Override
	public double findBookPriceByIsbn(String isbn) {
		// TODO Auto-generated method stub
		String sql = "SELECT price FROM book WHERE isbn = ?";

		return jdbcTemplate.queryForObject(sql, Double.class, isbn);
	}

	// 查余额
	@Override
	public double findBalanceById(int id) {
		// TODO Auto-generated method stub
		String sql = "SELECT balance FROM accounts WHERE id = ?";

		return jdbcTemplate.queryForObject(sql, Double.class, id);
	}

	// 查库存
	@Override
	public int findStockByIsbn(String isbn) {
		// TODO Auto-generated method stub
		String sql = "SELECT stock FROM book_stock WHERE isbn = ?";
		return jdbcTemplate.queryForObject(sql, Integer.class, isbn);
	}

	@Override
	public void updateBookStock(String isbn) {
		// TODO Auto-generated method stub
		int stock = findStockByIsbn(isbn);

		// 如果库存不足则抛出异常
		if (stock == 0) {
			throw (new BookStockException("库存不足！"));
		}

		String sql = "UPDATE book_stock SET stock = stock - 1 WHERE isbn = ?";
		jdbcTemplate.update(sql, isbn);

	}

	@Override
	public void updateUserBalance(int id, double price) {
		// TODO Auto-generated method stub

		// 如果余额不足则抛出异常
		double balance = findBalanceById(id);
		if (balance < price) {
			throw new UserBalanceException("余额不足！");
		}

		String sql = "UPDATE accounts SET balance = balance - ? WHERE id = ?";

		jdbcTemplate.update(sql, price, id);
	}

}

```





使用

```java
	@Test
	public void testBookShopService() {
		bookShopService.purchase(1, "1");
	}
```



## 5 事务的传播行为

- 当事务方法被另一个事务方法调用时, 必须指定事务应该如何传播. 例如: 方法可能继续在现有事务中运行, 也可能开启一个新事务, 并在自己的事务中运行.
- 事务的传播行为可以由传播属性指定. Spring 定义了 7  种类传播行为.

1. REQUIRED：如果有事务在运行，当前的方法就在这个事务内运行，否则，就启动一个新的事务，并在自己的事务内运行。
2. REQUIRED_NEW：当前的方法必须启动新的事务，并在它自己的事务内运行，如果有事务正在运行，应该将它挂起。

### 5.1 举例说明

仍以上面的书店例子为例：我们有一个买书的perchase()，该服务是一个单本购买服务。现在有一个新的需求，一个个顾客一次需要买多本书，也就是批量购买服务。批量购买服务会多次调用单本购买服务，那么问题来了，如果在一次批量购买中，已经完成了若干次单本购买，但是下一次单本购买出现了异常，那么应该回滚所有的购买事务，还是仅回滚当前购买失败的事务呢？换句话说，是一本都不买呢，还是能买多少买多少呢？这就是REQUIRED和REQUIRED_NEW的区别。默认为REQUIRED，表示如果当前方法整个是一个事务，如果回滚整个都回滚，也就是如果失败了则一本都不买。REQUIRED_NEW表示每个方法中的事务都启动一个新事务，失败时只回滚失败的那个事务，也就是能买多少，买多少。

1. 示例

   ```java
   /*
    * 结账服务
    */
   @Service("cashier")
   public class CashierImpl implements Cashier {
   
   	@Autowired
   	private BookShopService bookShopService;
   	
   	/*
   	 * 批量购买
   	 */
   	@Transactional
   	@Override
   	public void checkout(int userId, List<String> isbnList) {
   		// TODO Auto-generated method stub
   		for(String isbn : isbnList) {
   			bookShopService.purchase(userId, isbn);
   		}
   	}
   
   }
   ```

   当前账户余额和库存

   ![](Spring%20%E4%B8%AD%E7%9A%84%E4%BA%8B%E5%8A%A1%E7%AE%A1%E7%90%86.assets/1571810026459-1571811165897.png)

   ![](Spring%20%E4%B8%AD%E7%9A%84%E4%BA%8B%E5%8A%A1%E7%AE%A1%E7%90%86.assets/1571810026459-1571811181157.png)

   

   

2. 使用required

   ```java
   @Service("bookShopService")
   public class BookShopServiceImpl implements BookShopService {
   
   	@Autowired
   	BookShopDao bookShopDao;
   	
   	@Transactional // 默认使用REQUIRED，不启用新事务
   	@Override
   	public void purchase(int userid, String isbn) {
   		// TODO Auto-generated method stub
   		
   		bookShopDao.updateBookStock(isbn);
   		
   		double price = bookShopDao.findBookPriceByIsbn(isbn);
   		
   		bookShopDao.updateUserBalance(userid, price);
   	}
   
   }
   ```

   使用批量购买

   ```java
   // 测试结账服务
   	@Test
   	public void testCashierService() {
   		List<String> isbnList = new LinkedList<String>();
   		isbnList.add("2");
   		isbnList.add("1");
   		
   		// id为1的用户，购买isbn为1、2的书各一本
   		cashier.checkout(1, isbnList);
   	}
   
   ```

   结果：

   ![](Spring%20%E4%B8%AD%E7%9A%84%E4%BA%8B%E5%8A%A1%E7%AE%A1%E7%90%86.assets/1571810187147-1571811243872.png)

   ![](Spring%20%E4%B8%AD%E7%9A%84%E4%BA%8B%E5%8A%A1%E7%AE%A1%E7%90%86.assets/1571810207238-1571811254506.png)

   表示购买成功。

   

   再次运行测试程序，id为1的用户，购买isbn为1、2的书各一本。

   结果：

   > com.husthuangkai.spring.tx.UserBalanceException: 余额不足！
   > ....

   出现此异常，查看余额和库存：

   ![1571811398122](Spring%20%E4%B8%AD%E7%9A%84%E4%BA%8B%E5%8A%A1%E7%AE%A1%E7%90%86.assets/1571811398122.png)

   ![1571811404881](Spring%20%E4%B8%AD%E7%9A%84%E4%BA%8B%E5%8A%A1%E7%AE%A1%E7%90%86.assets/1571811404881.png)

   可见，默认情况下，使用REQUIRED参数，checkout（）是一整个事务，purchase（）不启用新事务，当发生异常时，checkout（）回滚。

   

3. 使用REQUIRED_NEW

   ```java
   @Service("bookShopService")
   public class BookShopServiceImpl implements BookShopService {
   
   	@Autowired
   	BookShopDao bookShopDao;
       
   	/*
   	 *使用 propagation指明事务的传播属性，默认为REQUIRED
   	 *REQUIRED：使用调用方法的事务，当异常发生时，回滚调用方法
   	 *REQUIRES_NEW：本方法要求启用新事务，当本方法发生异常时，仅回滚本事务
   	 */
   	// @Transactional // 默认使用REQUIRED，不启用新事务
   	@Transactional(propagation = Propagation.REQUIRES_NEW) // 指明使用REQUIRES_NEW，需要启用新事务
   	@Override
   	public void purchase(int userid, String isbn) {
   		// TODO Auto-generated method stub
   		
   		bookShopDao.updateBookStock(isbn);
   		
   		double price = bookShopDao.findBookPriceByIsbn(isbn);
   		
   		bookShopDao.updateUserBalance(userid, price);
   	}
   
   }
   ```

   运行测试程序：

   ```java
   // 测试结账服务
   	@Test
   	public void testCashierService() {
   		List<String> isbnList = new LinkedList<String>();
   		isbnList.add("2");
   		isbnList.add("1");
   		
   		// id为1的用户，购买isbn为1、2的书各一本
   		cashier.checkout(1, isbnList);
   	}
   ```

   结果：

   > com.husthuangkai.spring.tx.UserBalanceException: 余额不足！
   > ...
   >

   出现余额不足异常，查看余额和库存：

   ![1571813790298](Spring%20%E4%B8%AD%E7%9A%84%E4%BA%8B%E5%8A%A1%E7%AE%A1%E7%90%86.assets/1571813790298.png)

   ![1571813797605](Spring%20%E4%B8%AD%E7%9A%84%E4%BA%8B%E5%8A%A1%E7%AE%A1%E7%90%86.assets/1571813797605.png)

   结果是购买了一本书，表明purchase()启动了新的事务，发生异常时，perchase()回滚，checkout()不回滚。



## 6 事务的其他属性

### 6.1 事务的隔离级别

使用isolation指定事务的隔离级别，最常用的是READ_COMMITED。
READ_COMMITED：读已提交

### 6.2 事务的回滚属性

默认情况下，Spring的声明式事务对所有的运行时异常进行回滚，也可以通过对应的属性进行设置，通常情况下使用默认值即可。

### 6.3 事务的只读属性

使用readOnly指定事务是否只读，表示这个事务只读取数据但不更新数据，这样可以帮助数据库引擎优化事务。

### 6.4 事务的超时属性

使用timeout指定事务的时间限制，如果超时则回滚。



## 7 使用配置文件配置事务管理器

```xml
	<!-- 配置 bean -->
	<bean id="bookShopDao" class="com.atguigu.spring.tx.xml.BookShopDaoImpl">
		<property name="jdbcTemplate" ref="jdbcTemplate"></property>
	</bean>
	
	<bean id="bookShopService" class="com.atguigu.spring.tx.xml.service.impl.BookShopServiceImpl">
		<property name="bookShopDao" ref="bookShopDao"></property>
	</bean>
	
	<bean id="cashier" class="com.atguigu.spring.tx.xml.service.impl.CashierImpl">
		<property name="bookShopService" ref="bookShopService"></property>
	</bean>
	
	<!-- 1. 配置事务管理器 -->
	<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		<property name="dataSource" ref="dataSource"></property>
	</bean>
	
	<!-- 2. 配置事务属性 -->
	<tx:advice id="txAdvice" transaction-manager="transactionManager">
		<tx:attributes>
			<!-- 根据方法名指定事务的属性 -->
			<tx:method name="purchase" propagation="REQUIRES_NEW"/>
			<tx:method name="get*" read-only="true"/>
			<tx:method name="find*" read-only="true"/>
			<tx:method name="*"/>
		</tx:attributes>
	</tx:advice>
	
	<!-- 3. 配置事务切入点, 以及把事务切入点和事务属性关联起来 -->
	<aop:config>
		<aop:pointcut expression="execution(* com.atguigu.spring.tx.xml.service.*.*(..))" 
			id="txPointCut"/>
		<aop:advisor advice-ref="txAdvice" pointcut-ref="txPointCut"/>	
	</aop:config>
```

