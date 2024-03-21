# Spring

轻量级的开源框架，用于提高开发效率与系统的可维护性

Spring Framework 可以方便地集成第三方模块，无需重新造轮子，开箱即用

## 主要模块

![Spring5.x主要模块](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/20200831175708.png)

- Core Container：核心模块，提供 IoC 依赖注入功能，其他功能基本都需要依赖该模块
- AOP：提供面向切面编程的功能
- Data Access/Integration：提供对数据库访问的抽象 JDBC(适配器模式)，Java 只需要与 JDBC 交互，而不需要关注底层的数据库实现
- Web：提供 Web 相关功能的实现(MVC、Socket)
- Messaging：Spring 4.0 引入，负责为 Spring 框架集成基础报文传送应用
- Spring Test：Spring 团队提倡测试驱动开发(TDD)，通过 IoC 简化了单元测试与集成测试，Spring Test 模块对主流测试框架(JUnit)都有较好的支持

![Spring 各个模块的依赖关系](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/20200902100038.png)

### Spring vs Spring MVC vs Spring Boot

- Spring 即 Spring Framework，提供了核心的 Core Container 模块，其他功能大多需要依赖该模块提供的 IoC 功能
- Spring MVC 为 Spring 中的一个重要模块，主要用于快速构建 MVC 架构的 Web 程序
- Spring Boot 简化了 Spring 开发所需的配置(需要使用 XML/Java 对用到的功能显式配置)，实际的开发仍由其他模块实现(Spring MVC 构建 MVC 架构)

## Spring IoC

控制反转的设计思想：程序创建对象 → 框架创建对象

- 控制：对象的创建/管理
- 反转：控制权交给外部环境(框架、容器)

如类 A 依赖于类 B，则普通的开发方式：

```java
public staic void main(){
    A a = new B();
}
```

使用 IoC 将创建交由框架后，程序只需要声明使用，从 IoC 容器中获取即可：

```java
public static void main(){
    A a;
}
```

通过 IoC，程序虽然不能控制 A 的实例对象生成，但也不再需要考虑对象的管理

![IoC 图解](https://oss.javaguide.cn/java-guide-blog/frc-365faceb5697f04f31399937c059c162.png)

### 优点

- 降低对象之间的耦合度：程序只需要声明对象，而不需要关注实例的创建(不需要在代码中 new 出实现类)，由 IoC 负责对象的生成，可以方便地替换实例对象
- 资源易于管理：Spring 通过单例表注册表实现单例模式

==由 IoC 容器维护对象之间的依赖关系，并完成对象的注入，可以简化应用的开发，业务代码无需关注复杂的依赖关系==，只需要配置好配置文件/注入后就可以从 IoC 容器中获取对象

如 Service 类中可能依赖很多其他的类，若由程序负责创建，则需要管理所有依赖的底层类的构造函数，而使用 IoC 则只需要配置完成后直接引用即可，大大降低开发难度与提高可维护性

IoC 容器是实现 Spring IoC 的载体，==本质是一个 Map==，用于存放各种对象 

### IoC vs DI

IoC → 设计思想

DI(依赖注入) → 一种 IoC 实现

### Spring Bean

Bean 是被 IoC 容器管理的，被封装过的 Object 对象

通过配置元数据声明 IoC 容器应管理哪些对象，配置元数据可以为 xml，注解，配置类

```xml
<bean id="" class="">
	<constructor-arg value=""/>
</bean>
```

![](https://docs.spring.io/spring-framework/reference/_images/container-magic.png)

在 Spring 中使用 xml 文件进行配置，bean 较多的情况下配置繁琐且难以管理，故产生了 Spring Boot 进行优化，只需要使用注解即可将对象声明为 bean

#### 注解配置

- @Component：通用注解，能够将任意类标记为 Spring 组件，其他用于类的注解都由此扩展
- @Repository：作用于持久层，注解作为 DAO 对象的类，具有将数据库操作抛出的原生异常翻译转化为 Spring 持久层异常的功能
  - @Repository = @Component + 数据库异常转换
- @Service：作用于业务逻辑层，只起标注该类为逻辑层的作用
- @Controller：用于 Spring MVC 的注解，具有将请求进行转发、重定向的功能
  - @Controller = @Component + 请求路由处理
- @Configuration：声明为配置类，可以在其中声明多个 @Bean 方法
  - @Bean 方法只有用于 @Configuration 下时才为单例模式

#### @Component vs @Bean

- 作用域：@Component 作用于类，@Bean 作用于方法
- 注入方式：
  - @Component ==通过类路径扫描==来自动侦测以及自动装配到 Spring 容器中，可以用 @ComponentScan 注解定义从要扫描的路径中找出标识需要装配的类自动装配到 bean 容器中
  - @Bean 在被标注的方法中定义产生 bean，@Bean 告知 Spring 方法返回的对象是某个类(以方法名或 @name 为对象名)的实例，应该在需要时返回
- 灵活性：@Bean 的灵活性更高，可以返回动态的 bean，且由 @Bean 注入的对象不需要源码，故引入第三方库中的类时只能使用 @Bean；@Component 需要通过类路径扫描来将对象装配到容器中

使用 @Bean 注解：

```java
@Configuration
public class AppConfig {
    @Bean
    public AService aService() {
        return new AServiceImpl();
    }
}
```

等价于：

```xml
<beans>
	<bean id="aService" class="com.acme.AServiceImpl"/>
</beans>
```

@Component 无法实现：

```java
@Bean
public OneService getService(status) {
    case(status) {
        when 1:
        	return new serviceImpl1();
        when 2:
        	return new serviceImpl2();
        when 3:
        	return new serviceImpl3();
    }
}
```

#### Lite Mode vs Full Mode

@Bean 注解的方法只有在 @Configuration(proxyBeanMehtods = true) 的类中，注入容器的 bean 才为单例模式，即 Full Mode，否则为 Lite Mode

- Lite Mode：==配置类的 bean 对象不会被 CGLIB 增强==，@Bean 方法之间无法声明依赖，每个 @Bean 方法只是特定 bean 引用的工厂方法，没有特殊的运行时语义
  - bean 注入容器的是实例本身，无需生成额外的 CGLIB 对象，提高运行效率，==降低启动时间==
  - 配置类可以用作普通类，@Bean 方法可以为 private final
  - 由于 @Bean 方法无法声明 bean 之间的依赖，故不应在 @Bean 方法中调用其他的 @Bean 方法，应该将对应的依赖放入方法入参中
    - 在 @Bean 方法中调用其他 @Bean 会==生成新的实例而非使用 IoC 容器内的单例==
- Full Mode：@Bean 的交叉方法引用会被重定向到容器的生命周期管理，可以更方便管理 Bean 依赖
  - bean 对象为==单例模式==，即调用的一定是容器内的 bean
  - CGLIB 增强需要额外生成 CGLIB 子类注入容器，增加开销，影响启动时间
  - CGLIB 基于继承实现，故 @Bean 方法不能为 private final

#### bean 注入

通过注解注入 bean：

- @Autowired：Spring 内置注解，默认注入方式为 byType，当 bean 存在多个类型(接口有多个实现类)时，改为 byName
  - 可以通过 @Qualifier 指定名称
  - 应用范围更广，能作用于构造函数、方法、字段、参数
- @Resource：JDK 内置注解，默认注入方式为 byName，当不存在名称时改为 byType
  - 通过 @Resource(name = "") 指定名称
  - 主要用于字段和方法
- @Inject

#### 作用域

通过 xml 的 scope 字段或 @Scope 注解设置 bean 的作用域：

- singleton：默认的单例模式，IoC 容器中只有一个实例
- prototype：每次获取 bean 都会生成新的实例
- 仅 Web 可用：
  - request：每次请求生成新的 bean 实例
  - session：bean 实例只在一次 session 中可用
  - websocket：在一次 socket 连接中可用
  - application：每次应用重启都会生成新的 bean 实例

#### 线程安全

bean 的线程安全取决于其作用域和状态，大多数 bean 都是无状态的(没有可变的成员变量)，这类 bean 是线程安全的

- singleton：单例模式下会存在竞争，也会有线程安全问题
  - 避免使用可变的成员变量
  - 将可变的成员变量存储于 ThreadLocal 中
- prototype：每个线程都有自己的实例，不存在竞争，线程安全

#### 生命周期

1. 定义：容器通过配置文件得到 bean 的定义
2. 实例化：利用反射生成 bean 实例
3. 元数据：若有元属性的配置(实现了相关接口)，通过属性的 set 方法设置
   - BeanNameAware → setBeanName()
   - BeanClassLoaderAware → setBeanClassLoader()
   - BeanFactoryAware → setBeanFactory
   - xxxAware → setxxx
4. 初始化前：存在和加载 bean 的容器相关的 BeanPostProcessor 对象，执行 postProcessBeforeInitialization() 方法
5. 初始化：执行配置文件中定义的 ini-method
6. 初始化后：存在和加载 bean 的容器相关的 BeanPostProcessor 对象，执行 postProcessAfterInitialization() 方法
7. 销毁前：若实现了 DisposableBean 接口，执行 destroy() 方法
8. 销毁：执行配置文件中的 destroy-method

![Spring Bean 生命周期](https://images.xiaozhuanlan.com/photo/2019/b5d264565657a5395c2781081a7483e1.jpg)

## Spring AOP

面向切面编程，将业务模块中与业务无关的共同调用的逻辑(事务管理、业务管理、权限控制)封装起来，减少系统的重复代码、降低模块间耦合度

Spring AOP 基于动态代理实现，当被代理的对象实现了接口，则使用 JDK Proxy 创建代理对象，否则利用 CGLIB 创建被代理对象的子类

![SpringAOPProcess](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/230ae587a322d6e4d09510161987d346.jpeg)

### 基本概念

- 目标(Target)：被增强的对象
- 代理(Proxy)：向目标对象应用增强(Advice)后创建的代理对象
- 连接点(JoinPoint)：目标对象的所属类中，定义的所有方法均为连接点
- 切入点(Pointcut)：被切面增强的连接点
- 增强(Advice)：用于增强的逻辑，即拦截到目标对象后进行的操作
- 切面(Aspect)：切入点 + 增强
- 织入(Weaving)：将增强应用到目标对象，进而生成代理对象的操作