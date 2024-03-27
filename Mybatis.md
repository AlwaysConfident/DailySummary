# MyBatis

MyBatis 框架用于简化 Java 程序与数据库连接的操作

## JDBC

JDBC 操作数据库：

```java
// 加载配置文件
Properties pro = new Properties();
pro.load(new FileReader("resource/jdbc.properties"));
// 获取配置文件中连接数据库的信息
String url = pro.getProperty("url");
String user = pro.getProperty("user");
String password = pro.getProperty("password");
String driver = pro.getProperty("driver");
// 加载数据库驱动
Class.forName(driver);
// 创建数据库连接
Connection conn = DriverManager.getConnection(url, user, password);
// sql 语句
String sql = "select * from a where username = ?";
// 创建执行 sql 的对象
ps = conn.prepareStatement(sql);
// 给 ? 复制
ps.setString(1, username);
// 执行 sql
ResultSet rs = ps.executeQuery;
// 查询出数据，则返回
if(rs.next()) {
    Admin admin = new Admin();
    admin.setUsername(rs.getString("username"));
    admin.setPassword(rs.getString("password"));
    return admin;
} else {
    return null;
}
```

- 每次数据库操作都需要创建 connection、Statement 等连接相关的对象，且需要手动关闭与销毁
- 查询结果 ResultSet 无法自动转换为实体对象，需要手动 set
- 业务代码与数据库连接操作耦合

Mybatis 内部封装 jdbc，开发者只需通过 xml 或注解配置 sql 语句，由 Mybatis 负责加载驱动、创建连接、映射实体类等繁琐操作

## SqlSessionFactory

每个基于 MyBatis 的应用都是以一个 SqlSessionFactory 的实例为核心

XML配置文件 或 Configuration 实例 → SqlSessionFactoryBuilder → SqlSessionFactory → SqlSession

### XML 配置文件

利用 Mybatis 中的 Resource 工具类读取 XML，通过 SqlSessionFactoryBuilder 创建 SqlSessionFactory 实例

```java
String resource = "org/mybatis/example/mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource);
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
```

XML 配置文件包含 MyBatis 系统的核心配置：

- 数据库连接实例的数据源
- 事务管理器

### Configuration 配置类

通过 Java 代码实现自定义的配置构建器，不支持复杂的映射关系(嵌套联合映射)，同名的 XML 配置文件具有更高的优先级

```java
DataSource dataSource = BlogDataSourceFactory.getBlogDataSource();
TransactionFactory transactionFactory = new JdbcTransactionFactory();
Environment environment = new Environment("development", transactionFactory, dataSource);
Configuration configuration = new Configuration(environment);
configuration.addMapper(BlogMapper.class);
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(configuration);
```

## SqlSession

SqlSession 提供在数据库执行 SQL 命令所需的所有方法，即可以通过 SqlSession 实例执行已映射的 SQL 语句

### XML 配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "https://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="org.mybatis.example.BlogMapper">
  <select id="selectBlog" resultType="Blog">
    select * from Blog where id = #{id}
  </select>
</mapper>
```

在命名空间 "org.mybatis.example.BlogMapper" 中定义了名为 "selectBlog" 的映射语句，由此可以通过全限定名 "org.mybatis.example.BlogMapper.selectBlog" 调用映射语句：

```java
try(SqlSession session = sqlSessionFactory.openSession()) {
    Blog blog = (Blog)session.selectOne("org.mybatis.example.BlogMapper.selectBlog", 101);
}
```

### Mapper

通过 Mapper 可以令代码更简洁，且类型安全，无需担心可能出错的字符串字面值以及强制类型转换

XML 中的全限定命名可以像 Java 对象一样直接映射到在命名空间中同名的映射器类，并将已映射的 select 语句匹配到对应名称、参数、返回类型的方法，故可以在对应的映射器接口调用方法：

````java
try(SqlSession session = sqlSessionFactory.openSession()) {
    BlogMapper mapper = session.getMapper(BlogMapper.class);
    Blog blog = mapper.selectBlog(101);
}
````

### 基于 Java 注解配置

对于映射器类可以使用 Java 注解进行简单 SQL 语句的配置，能够令代码更加简洁，但在复杂 SQL 场景则会使 SQL 语句更加混乱

```java
public interface BlogMapper {
    @Select("SELECT * FROM blog WHERE id = #{id}")
    Blog selectBlog(int id);
}
```

## 作用域和生命周期

依赖注入框架能够创建线程安全、基于事务的 SqlSession 和映射器，并将其注入 bean，故两者的生命周期不需要人为控制

- SqlSessionFactoryBuilder：只用于创建 SqlSessionFactory 实例，可以创建、使用、丢弃，由于 SqlSessionFactory 一般只需要实例化一次，故 SqlSessionFactoryBuilder 使用后就可以回收，保证 XML 解析资源被释放
  - 作用域：方法作用域，保证使用后就立即回收
- SqlSessionFactory：用于创建执行 SQL 的 SqlSession，被实例化后应在程序的整个生命周期中存在
  - 作用域：应用作用域，可以通过单例模式确保只有一个 SqlSessionFactory
- SqlSession：每个线程都有自己的 SqlSession 实例，且 SqlSession 自身是线程不安全的，故 SqlSession 不应被其他线程共享
  - 作用域：方法/请求作用域，