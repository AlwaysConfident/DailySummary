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
  - 作用域：方法/请求作用域，不应将 SqlSession 实例的引用放在类的静态域，甚至一个类的实例变量也不行，也不应将 SqlSession 实例引用放在任何类型的托管作用域中(Servlet 框架中的 HttpSession)
- mapper：绑定映射语句的接口，mapper 接口实例从 SqlSession 中获取，且不需要被显示关闭
  - 作用域：方法作用域，理论上应与请求的 SqlSession 相同，但 mapper 在调用方法结束后就可以丢弃，由此避免在作用域中管理过多的 SqlSession 资源

## XML 配置

MyBatis 配置文件会影响 MyBatis 行为的设置和属性信息

### 属性

属性可以在外部进行配置，且可以进行动态替换("${}")

若一个属性在多处进行配置：

- 首先读取在 properties 元素中指定的属性
- 然后根据 properties 元素中的 resource 属性读取类路径下属性文件，并==覆盖之前读过的同名属性==
- 最后读取作为方法参数传递的属性，并==覆盖之前读过的同名属性==

即 方法参数 > resource 路径下配置文件 > 默认配置文件

#### 默认属性

从 MyBatis 3.4.2 开始，可以通过开启特性为占位符指定默认值：

```xml
<properties resource="org/mybatis/example/config.properties">
  <!-- ... -->
  <property name="org.apache.ibatis.parsing.PropertyParser.enable-default-value" value="true"/> <!-- 启用默认值特性 -->
</properties>

<dataSource type="POOLED">
  <!-- ... -->
  <property name="username" value="${username:ut_user}"/> <!-- 如果属性 'username' 没有被配置，'username' 属性的值将为 'ut_user' -->
</dataSource>
```

当 ":" 字符被占用时，需要替换为其他特殊字符

```xml
<properties resource="org/mybatis/example/config.properties">
  <!-- ... -->
  <property name="org.apache.ibatis.parsing.PropertyParser.default-value-separator" value="?:"/> <!-- 修改默认值的分隔符 -->
</properties>
```

### 设置

MyBatis 中极为重要的调整设置，会改变 MyBatis 的运行时行为

- cacheEnabled：全局性地开启或关闭所有映射器配置文件中已配置的任何缓存。
- lazyLoadingEnabled：延迟加载的全局开关。当开启时，所有关联对象都会延迟加载。 特定关联关系中可通过设置 `fetchType` 属性来覆盖该项的开关状态。
- useGeneratedKeys：允许 JDBC 支持自动生成主键，需要数据库驱动支持。如果设置为 true，将强制使用自动生成主键。尽管一些数据库驱动不支持此特性，但仍可正常工作（如 Derby）。
- defaultExecutorType： 配置默认的执行器。SIMPLE 就是普通的执行器；REUSE 执行器会重用预处理语句（PreparedStatement）； BATCH 执行器不仅重用语句还会执行批量更新。

### 类型别名

别名可为 Java 类型设置缩写，只用于减少 XML 配置文件中冗余的全限定类名

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

也可以指定一个包名，MyBatis 会在包名下搜索需要的 bean，没有注解时默认使用 bean 的首字母小写类名

```xml
<typeAliases>
  <package name="domain.blog"/>
</typeAliases>
```

```java
@Alias("abc")
public class Author {
    
}
```

### 类型处理器

MyBatis 在设置预处理语句中的参数或从结果集中取出一个值时，会使用类型处理器将获取到的值转换成 Java 类型

可以通过重写或新增类型处理器来处理不支持的或非标准的类型：

- 实现 TypeHandler 接口
- 继承 BaseTypeHandler 类

使用自定义的类型处理器会覆盖默认的对应类型的处理器

MyBatis 不会通过检测数据库元信息来决定使用哪种处理器，因为 ==MyBatis 只有在语句执行时才清楚数据类型==，所以必须在参数和结果映射中指明字段类型

MyBatis 获取类型处理器的类型：配置文件中 typeHandler 元素的 javaType 属性 > 类型处理器类上的注解 @MappedTypes > 类型处理器的泛型

#### 枚举类型

映射枚举类型需要使用 EnumTypeHandler(默认) 或 EnumOrdinalTypeHandler

EnumTypeHandler 会处理任意继承了 Enum 的类，默认情况下将 Enum 值转换为其字面量，通过指定 typeHandler 的 javaType 属性更改转换的类型

### 对象工厂

MyBatis 通过对象工厂生成结果对象的实例，默认的对象工厂只负责实例化目标类，通过无参构造或通过存在的参数映射调用带有参数的构造方法

继承 DefaultObjectFactory 以实现自定义的对象工厂 

### 插件

MyBatis 运行在映射语句执行过程中的某一点进行拦截调用，默认情况下允许使用插件来拦截的方法调用：

- Executor (update, query, flushStatements, commit, rollback, getTransaction, close, isClosed)
- ParameterHandler (getParameterObject, setParameters)
- ResultSetHandler (handleResultSets, handleOutputParameters)
- StatementHandler (prepare, parameterize, batch, update, query)

只需要实现 Interceptor 接口并指定需要拦截的方法签名即可使用插件

```java
// ExamplePlugin.java
@Intercepts({@Signature(
  type= Executor.class,
  method = "update",
  args = {MappedStatement.class,Object.class})})
public class ExamplePlugin implements Interceptor {
  private Properties properties = new Properties();

  @Override
  public Object intercept(Invocation invocation) throws Throwable {
    // implement pre processing if need
    Object returnObject = invocation.proceed();
    // implement post processing if need
    return returnObject;
  }

  @Override
  public void setProperties(Properties properties) {
    this.properties = properties;
  }
}
```

### 环境配置

MyBatis 可以配置成多种环境，将 SQL 映射应用于多种数据库中，便于区分开发、测试、生产环境，也有利于数据库的迁移

虽然 MyBatis 能连接多个数据库，但生成的 SqlSession 实例只能对应其中一个

#### 事务管理器

事务管理器是环境配置的一个属性，在 MyBatis 中有两种配置管理器：

- JDBC：直接使用内部封装的 JDBC 的提交和回滚功能，依赖连接来管理事务作用域，默认情况下在关闭连接后会自动提交
- MANAGED：不会提交和回滚连接，由容器进行事务管理

Spring 框架会使用自带的事务管理器覆盖 MyBatis 的配置

#### 数据源

MyBatis 使用标准的 JDBC 数据源接口进行 JDBC 连接对象资源的配置，有三种内置的数据源类型：

- UNPOOLED：不使用连接池，每次请求都会打开与关闭一个连接
  - 适用于数据库连接可用性要求不高的简单程序，性能表现依赖于数据库，故适用于连接池不重要的数据库
- POOLED：使用"池"的概念组织 JDBC 连接对象，避免创建新的连接实例所需的初始化和认证时间，能够快速响应并发请求
- JNDI：EJB 或应用服务器这类容器可以集中或在外部配置数据源，然后放置一个 JNDI 上下文的数据源引用

## XML 映射器

MyBatis 通过简单的 XML 映射语句减少了大量的 JDBC 代码

### select

每个插入、更新、删除操作之间，通常会执行多个查询操作

### insert/update/delete

insert 语句支持自定义主键生成方法

可以传入集合，通过 foreach 标签进行批量处理

### sql

用于定义可重用的 SQL 代码片段，以便在其他语句中使用，参数可以在编译期确定：

```xml
<sql id="userColumns"> ${alias}.id,${alias}.username,${alias}.password </sql>
```

也可以在不同的 include 元素中定义不同的参数值：

```xml
<select id="selectUsers" resultType="map">
  select
    <include refid="userColumns"><property name="alias" value="t1"/></include>,
    <include refid="userColumns"><property name="alias" value="t2"/></include>
  from some_table t1
    cross join some_table t2
</select>
```

### 参数

简单的使用场景：

```xml
<select id="selectUsers" resultType="User">
  select id, username, password
  from users
  where id = #{id}
</select>
```

原始类型或简单数据类型在没有其他属性时会直接使用值作为参数

传入复杂对象：

```xml
<insert id="insertUser" parameterType="User">
  insert into users (id, username, password)
  values (#{id}, #{username}, #{password})
</insert>
```

MyBatis 会查找传入的 User 中的 id、username、password 属性，并将值传入预处理语句的参数中

参数通过设置 jdbcType、javaType、typeHandler 等属性自定义参数及其处理方式，但 允许 null 且会使用 null 的参数必须指定 jdbcType

#### 字符串替换(${} vs #{})

默认情况下，使用 #{} 时，MyBatis 会创建 PreparedStatement 参数占位符，并通过占位符按序安全地设置参数

如 #{item.name} 会使用反射从参数对象中获取 item 对象的 name 值，相当于 param.getItem().getName()

```xml
<select id="selectPerson" parameterType="int" resultType="hashmap">
  SELECT * FROM PERSON WHERE ID = #{id}
</select>
```

上述 SQL 会创建一个预处理语句：

```sql
SELECT * FROM PERSON WHERE ID = ?
```

${} 是 properties 文件中的变量占位符，可以用于标签属性值和 sql 内部，属于原样文本替换，可以替换任意内容，不会对传参进行修改或转义

如用于排序的字段 ${orderColumn} 可以为 `name` 、`name desc`、`name, sex asc`，从而实现==更为灵活==的排序，但在接收用户输入会导致潜在的 sql 注入攻击

#{} 通过预编译 SQL，将传入的参数都视为字符串，即使传入 sql 注入语句也不会被执行，如：

```sql
SELECT * FROM PERSON WHERE ID = 1 OR ID > 0
```

会被编译为：

```sql
SELECT * FROM PERSON WHERE ID = '1 OR ID > 0'
```

### 结果映射

resultMap 极大地简化了 JDBC 的 ResultSets 数据提取代码，且在一些情形下允许进行一些 JDBC 不支持的操作

通过 resultType="" 使用全限定类名将 ResultSets 映射到 bean，MyBatis 通过自动生成的 resultMap 根据属性名映射列到 JavaBean 的属性，而不需要进行显示配置

#### 复杂的结果映射

- `constructor` - 用于在实例化类时，注入结果到构造方法中

  - `idArg` - ID 参数；标记出作为 ID 的结果可以帮助提高整体性能

  - `arg` - 将被注入到构造方法的一个普通结果
- `id` – 一个 ID 结果；标记出作为 ID 的结果可以帮助提高整体性能
- `result` – 注入到字段或 JavaBean 属性的普通结果
- `association` – 一个复杂类型的关联；许多结果将包装成这种类型，表示从属关系
- 嵌套结果映射 – 关联可以是 `resultMap` 元素，或是对其它结果映射的引用
- `collection` – 一个复杂类型的集合
- 嵌套结果映射 – 集合可以是 `resultMap` 元素，或是对其它结果映射的引用
- `discriminator` – 使用结果值来决定使用哪个 `resultMap`
  - `case` – 基于某些值的结果映射
    - 嵌套结果映射 – `case` 也是一个结果映射，因此具有相同的结构和元素；或者引用其它的结果映射

#### id & result

```xml
<id property="id" column="post_id"/>
<result property="subject" column="post_subject"/>
```

id 和 result 元素都将一个列的值映射到一个简单数据类型的属性或字段

两者的唯一不同是，id 元素对应的属性会被标记为对象的标识符，在比较对象实例时使用，可以提交整体性能，尤其是进行缓存和嵌套结果映射时

#### 关联

association 元素处理"有一个"类型的关系，关联结果映射需要告知 MyBatis 如何加载关联：

- 嵌套 Select 查询：通过执行另外一个 SQL 映射语句加载期望的复杂类型
- 嵌套结果映射：使用嵌套的结果映射处理连接结果的重复子集

##### 嵌套 select 查询

```xml
<resultMap id="blogResult" type="Blog">
  <association property="author" column="author_id" javaType="Author" select="selectAuthor"/>
</resultMap>

<select id="selectBlog" resultMap="blogResult">
  SELECT * FROM BLOG WHERE ID = #{id}
</select>

<select id="selectAuthor" resultType="Author">
  SELECT * FROM AUTHOR WHERE ID = #{id}
</select>
```

select = "selectAuthor" 表示应该使用 selectAuthor 语句加载 author 属性，即当 selectBlog 时，会自动通过 selectAuthor 加载嵌套属性

这种方式简单，但会造成 =="N + 1 查询问题"==：

- 1 → 一个查询结果列表
- N → 为查询结果列表的 N 行都执行 select 语句加载嵌套信息

导致海量的 SQL 语句被执行

MyBatis 对这样的查询进行延迟加载，因此可以将大量语句同时运行的开销分散，但在加载结果列表后立刻遍历列表以获取嵌套数据时会触发所有延迟加载查询，严重影响性能

##### 关联的嵌套结果映射
