# Spring Jdbc 支持

- 为了使 JDBC 更加易于使用, Spring 在 JDBC API 上定义了一个抽象层, 以此建立一个 JDBC 存取框架.
- 作为 Spring JDBC 框架的核心, JDBC 模板的设计目的是为不同类型的 JDBC 操作提供模板方法. 每个模板方法都能控制整个过程, 并允许覆盖过程中的特定任务. 通过这种方式, 可以在尽可能保留灵活性的情况下, 将数据库存取的工作量降到最低.

## 1. 使用JdbcTemplate

### 1.1 准备工作

1. 配置xml文件

   ```xml
   <!-- 1. 导入资源文件 -->
   	<context:property-placeholder location="classpath:db.properties"/>
   	
   <!-- 2. 配置C3P0数据源 -->
   	<bean id="dataSource"
   		class="com.mchange.v2.c3p0.ComboPooledDataSource">
   		<property name="user" value="${jdbc.user}"></property>
   		<property name="password" value="${jdbc.password}"></property>
   		<property name="jdbcUrl" value="${jdbc.jdbcUrl}"></property>
   		<property name="driverClass" value="${jdbc.driverClass}"></property>
   		<property name="initialPoolSize" value="${jdbc.initPoolSize}"></property>
   		<property name="maxPoolSize" value="${jdbc.maxPoolSize}"></property>
   	</bean>
   	
   <!-- 3. 配置 Spring 的 JdbcTemplate -->	
   	<bean id="jdbcTemplate" 
   	class="org.springframework.jdbc.core.JdbcTemplate">
   		<property name="dataSource" ref="dataSource"></property>
   	</bean>
   ```

2. 配置资源文件

   > jdbc.user=root
   > jdbc.password=123456
   > jdbc.jdbcUrl=jdbc:mysql://localhost:3306/taotao?useSSL=false&serverTimezone=UTC
   > jdbc.driverClass=com.mysql.cj.jdbc.Driver
   >
   > jdbc.initPoolSize=5
   > jdbc.maxPoolSize=10

   注意再mysql8.0版本之后，driverClass要配置为com.mysql.cj.jdbc.Driver，

   Url后要加上?useSSL=false&serverTimezone=UTC

3. 获取JdbeTemplate对象

   ```java
   private ApplicationContext ctx = null;
   	private JdbcTemplate jdbcTemplate;
   	
   	{
   		ctx = new ClassPathXmlApplicationContext("ApplicationContext.xml");
   		jdbcTemplate = (JdbcTemplate) ctx.getBean("jdbcTemplate");
   	}
   
   ```

### 1.2 使用JdbcTemplate

1. 单行更新

   ```java
   	// 单行更新
   	@Test
   	public void testUpdate() {
   		String sqlString = "UPDATE employees SET dept_id = ? WHERE id = ?";
   		jdbcTemplate.update(sqlString, 5, 11);
   	}
   ```

2. 批量更新

   ```java
   /*
   	 * 执行批量更新，批量INSERT，UPDATE，DELETE
   	 * 最后一个参数时Object[]的list类型，因为修改一条记录需要一个Object[]数组，那么多条就需要多个Object[]
   	 */
   	@Test
   	public void testBatchUpdate() {
   		String sqlString = "INSERT INTO employees(last_name, email, dept_id) VALUES(?,?,?)";
   		
   		List<Object[]> batchArgs = new ArrayList<Object[]>();
   		
           // 每个batchArgs表示一条语句
   		batchArgs.add(new String[] {"aa", "aa@hust.com", "10"});
   		batchArgs.add(new String[] {"bb", "bb@hust.com", "11"});
   		batchArgs.add(new String[] {"cc", "cc@hust.com", "12"});
   		batchArgs.add(new String[] {"dd", "dd@hust.com", "13"});
   		batchArgs.add(new String[] {"ee", "ee@hust.com", "14"});
   		
   		
   		jdbcTemplate.batchUpdate(sqlString, batchArgs);
   	}
   
   ```

3. 单条查询

   ```java
   /*
   	 * 获取单个列的值，或做统计查询
   	 * 使用jdbcTemplate.queryForObject(sql, Long.class);
   	 */
   	@Test
   	public void testQueryObject2() {
   		String sql = "SELECT count(id) FROM employees";
   		long count = jdbcTemplate.queryForObject(sql, Long.class);
   		System.out.println(count);
   	}
   
   /*
   	 * 从数据库中获取一条记录，实际得到对应的一个对象
   	 * 注意不是调用queryForObject(String sql, Class<Employee> requiredType, Object... args)方法
   	 * 而是要调用queryForObject(String sql, RowMapper<Employee> rowMapper, Object... args)
   	 * 1.其中RowMapper<Employee>指定如何去映射结果集的行，常用的实现类为BeanPropertyRowMapper<>
   	 * 2.使用SQL中的别名完成列名和属性名的映射，last_name lastName
   	 * 3.不支持级联属性，JdbcTemplate到底是一个JDBC的小工具，而不是ORM框架
   	 */
   	@Test
   	public void testQueryForObject() {
   		String sqlString = "SELECT id, last_name lastName, email FROM employees WHERE id = ?";
   		org.springframework.jdbc.core.RowMapper<Employee> rowMapper = new BeanPropertyRowMapper<>(Employee.class);
   		Employee employee = jdbcTemplate.queryForObject(sqlString, rowMapper, "11");
   		System.out.println(employee);
   	}
   ```

4. 批量查询

   ```java
   /*
   	 * 查到实体的集合
   	 * 注意调用的不是queryForList方法
   	 */
   	@Test
   	public void testQueryForList() {
   		String sql = "SELECT id, last_name lastName, email FROM employees WHERE id > ?";
   		RowMapper<Employee> rowMapper = new BeanPropertyRowMapper<Employee>(Employee.class);
   		List<Employee> employees = jdbcTemplate.query(sql, rowMapper, 10);
   		System.out.println(employees);
   	}
   ```

## 2 使用具名参数

### 2.1 准备工作

在配置文件中加入NamedParameterJdbcTemplate对象，其他同上

```xml
<!-- 配置NamedParameterJdbcTemplate,该对象可以使用具名参数 -->
	<bean id="namedParameterJdbcTemplate" 
		class="org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate">
		<constructor-arg ref="dataSource"></constructor-arg>
	</bean>
```



### 2.2 使用具名参数

```java
/*
	 * 可以为参数起名
	 * 1. 好处：若有多个参数，则不用再对应位置，直接对应参数名，便于维护
	 * 2. 缺点：较为麻烦
	 */
	@Test
	public void testNamedParameterJdbcTemplate() {
		String sql = "INSERT INTO employees(last_name, email, dept_id) VALUES(:ln,:email,:deptid)";
		
		Map<String, Object> paramMap = new HashMap<String, Object>();
		paramMap.put("ln", "FF");
		paramMap.put("email", "ff@cpm");
		paramMap.put("deptid", 2);
		
		namedParameterJdbcTemplate.update(sql, paramMap);
	}

/*
	 * 使用具名参数时，可以使用.NamedParameterJdbcTemplate.update(String sql, SqlParameterSource paramSource)方法进行操作
	 * 1.SQL语句中的参数名和类的属性一致
	 * 2.使用SqlParameterSource的BeanPropertySqlParameterSource实现类作为参数
	 * 3.BeanPropertySqlParameterSource可以直接包装一个对象，使用该对象的getter、setter方法
	 */
	@Test
	public void testNamedParameterJdbcTemplate2() {
		String sql = "INSERT INTO employees(last_name, email, dept_id) VALUES(:lastName,:email,:deptId)";
		
		Employee employee = new Employee();
		employee.setLastName("XUE");
		employee.setEmail("xue@com");
		employee.setDeptId(3);

		SqlParameterSource parameterSource = new BeanPropertySqlParameterSource(employee);
		namedParameterJdbcTemplate.update(sql, parameterSource);
	}
```

