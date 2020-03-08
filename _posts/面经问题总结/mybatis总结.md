### 1 快速入门mybatis

1. 添加mybatis依赖。

2. xml配置数据源。

3. 获取SqlSessionFactory

   ```java
   String resource = "org/mybatis/example/mybatis-config.xml";
   InputStream inputStream = Resources.getResourceAsStream(resource);
   SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
   ```

4. 从SqlSessionFactory获取SqlSession

   ```java
   // 方式1，使用这种方式即使使用xml配置也要创建BolgMapper接口
   try (SqlSession session = sqlSessionFactory.openSession()) {
     BlogMapper mapper = session.getMapper(BlogMapper.class);
     Blog blog = mapper.selectBlog(101);
   }
   // 方式2，这种方式直接指定命名空间，不需要创建BolgMapper接口
   Blog blog = (Blog) session.selectOne("org.mybatis.example.BlogMapper.selectBlog", 101);
   ```

5. 使用注解或xml映射sql语句。

   ```xml
   <mapper namespace="org.mybatis.example.BlogMapper">
     <select id="selectBlog" resultType="Blog">
       select * from Blog where id = #{id}
     </select>
   </mapper>
   ```

   ```java
   public interface BlogMapper {
     @Select("SELECT * FROM blog WHERE id = #{id}")
     Blog selectBlog(int id);
   }
   ```

6. 作用域和声明周期

   - SqlSessionFactoryBuilder：创建完SqlSessionFactory就丢弃。
   - SqlSessionFactory：应该是单例的，整个应用只有一个。
   - SqlSession：会话，线程不安全的，每次使用完关闭。
   - Mapper：局部作用域。



### 2 配置

1. 属性，使用properties，可以在其它配置中使用${}来取properties中的属性。

   ```xml
   <properties resource="org/mybatis/example/config.properties">
     <property name="username" value="dev_user"/>
     <property name="password" value="F2Fa3!33TYyg"/>
   </properties>
   ```

2. 设置，设置mybatis的一些重要参数

   ```xml
   <settings>
     <setting name="cacheEnabled" value="true"/>
     <setting name="lazyLoadingEnabled" value="true"/>
     <setting name="multipleResultSetsEnabled" value="true"/>
     <setting name="useColumnLabel" value="true"/>
     <setting name="useGeneratedKeys" value="false"/>
     <setting name="autoMappingBehavior" value="PARTIAL"/>
     <setting name="autoMappingUnknownColumnBehavior" value="WARNING"/>
     <setting name="defaultExecutorType" value="SIMPLE"/>
     <setting name="defaultStatementTimeout" value="25"/>
     <setting name="defaultFetchSize" value="100"/>
     <setting name="safeRowBoundsEnabled" value="false"/>
     <setting name="mapUnderscoreToCamelCase" value="false"/>
     <setting name="localCacheScope" value="SESSION"/>
     <setting name="jdbcTypeForNull" value="OTHER"/>
     <setting name="lazyLoadTriggerMethods" value="equals,clone,hashCode,toString"/>
   </settings>
   ```

3. 类型别名，可以为要映射的类指定一个简单的别名。

   - 通过xml设置

     ```xml
     <typeAliases>
       <typeAlias alias="Author" type="domain.blog.Author"/>
       <typeAlias alias="Blog" type="domain.blog.Blog"/>
       <typeAlias alias="Comment" type="domain.blog.Comment"/>
       <typeAlias alias="Post" type="domain.blog.Post"/>
       <typeAlias alias="Section" type="domain.blog.Section"/>
       <typeAlias alias="Tag" type="domain.blog.Tag"/>
     </typeAliases>
     ```

   - 通过注解设置

     ```java
     @Alias("author")
     public class Author {
         ...
     }
     ```

4. 类型处理器，指将java数据类型和db数据类型进行转化的处理器，mybatis中有默认的类型处理器，也可以自己实现覆盖，一个类型处理器的实现如下所示:

   ```java
   // ExampleTypeHandler.java
   @MappedJdbcTypes(JdbcType.VARCHAR)
   public class ExampleTypeHandler extends BaseTypeHandler<String> {
   
     @Override
     public void setNonNullParameter(PreparedStatement ps, int i, String parameter, JdbcType jdbcType) throws SQLException {
       ps.setString(i, parameter);
     }
   
     @Override
     public String getNullableResult(ResultSet rs, String columnName) throws SQLException {
       return rs.getString(columnName);
     }
   
     @Override
     public String getNullableResult(ResultSet rs, int columnIndex) throws SQLException {
       return rs.getString(columnIndex);
     }
   
     @Override
     public String getNullableResult(CallableStatement cs, int columnIndex) throws SQLException {
       return cs.getString(columnIndex);
     }
   }
   ```

   ```xml
   <!-- mybatis-config.xml -->
   <typeHandlers>
     <typeHandler handler="org.mybatis.example.ExampleTypeHandler"/>
   </typeHandlers>
   ```



### 3 xml映射文件

1. select

   ```xml
   <select id="selectPerson" parameterType="int" resultType="hashmap">
     SELECT * FROM PERSON WHERE ID = #{id}
   </select>
   ```

   #{id}相当于JDBC编程的时候使用的？占位符，然后使用id作为参数，这样可以防止sql注入。

2. 结果映射

   ```java
   <!-- 简单对象可以不映射 -->
   <select id="selectUsers" resultType="com.someapp.model.User">
     select id, username, hashedPassword
     from some_table
     where id = #{id}
   </select>
       
   <!-- 也可以使用resultMap映射 -->
   <resultMap id="userResultMap" type="User">
     <id property="id" column="user_id" />
     <result property="username" column="user_name"/>
     <result property="password" column="hashed_password"/>
   </resultMap>
   <select id="selectUsers" resultMap="userResultMap">
     select user_id, user_name, hashed_password
     from some_table
     where id = #{id}
   </select>
   ```

   