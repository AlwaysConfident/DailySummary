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

==Spring AOP 基于动态代理实现，属于运行时增强==，当被代理的对象实现了接口，则使用 JDK Proxy 创建代理对象，否则利用 CGLIB 创建被代理对象的子类

AspectJ AOP 是 Java 生态中的 AOP 框架，==基于字节码操作进行编译时增强==，比 Spring AOP 更强大也更复杂，适用于切面较多的场景

![SpringAOPProcess](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/230ae587a322d6e4d09510161987d346.jpeg)

### 基本概念

- 目标(Target)：被增强的对象
- 代理(Proxy)：向目标对象应用增强(Advice)后创建的代理对象
- 连接点(JoinPoint)：目标对象的所属类中，定义的所有方法均为连接点
- 切入点(Pointcut)：需要被增强的连接点
- 增强(Advice)：用于增强的逻辑，即拦截到目标对象后进行的操作
- 切面(Aspect)：切入点 + 增强
- 织入(Weaving)：将增强应用到目标对象，进而生成代理对象的操作

### AOP vs OOP

AOP 是 OOP 的一个补充，两者并不对立，OOP 难以处理多个对象中的公共行为，需要重复实现，且不利于扩展和维护，修改公共方法也需要修改业务代码

- AOP：将与具体业务逻辑无关，但被多个业务共享的代码/服务(事务管理、日志管理)通过动态代理或字节码操作进行提取封装，降低代码耦合度，提高可维护性
- OOP：将业务逻辑按照对象的属性和行为进行封装，通过继承、封装、多态等概念，实现代码的模块化和层次化，提高代码的可读性和可维护性

### AspectJ 通知类型

- Before：目标对象方法调用前
- After：目标对象方法调用后
- AfterReturning：目标对象方法调用完成，在返回结果后
- AfterThrowing：目标对象方法运行中抛出异常后触发，与 AfterReturning 互斥，即会抛出异常则不会正常返回
- Around：可以直接拿到目标对象以及要执行的方法，任意地在目标对象的方法调用前后增强，甚至不需要调用方法

### 切面执行顺序

- @Order 注解直接定义切面顺序，@Order(n) 值越小优先级越高
- 实现 Ordered 接口重写 getOrder 方法

## Spring MVC

MVC 的核心思想是将业务逻辑、数据、视图分离来组织代码

### 核心组件

- DispatcherServlet：核心的中央处理器，负责接收请求、分发、响应
- HandlerMapping：处理器映射器，URL → Handler，将请求用到的拦截器与 Handler 封装
- HandlerAdapter：处理器适配器，根据 HandlerMapping 映射的 Handler，适配执行对应的 Handler
- Handler：请求处理器，实际处理请求
- ViewResolver：视图解析器，将 Hander 返回的逻辑视图解析并渲染为真正的视图，传递给 DispatcherServlet 响应客户端

### 工作原理

![](https://img-blog.csdnimg.cn/87caa037d8584c559b014e7ca8e36a27.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5q2i5q2l5YmN6KGM,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

1. 客户端发送请求 → DispatcherServlet
2. DispatcherServlet → HandlerMapping 通过 URL 查找能够处理请求的 Handler，将拦截器与 Handler 一起封装返回
3. DispatcherServlet → HandlerAdapter 执行对应的 Handler
4. Handler → DispatcherServlet 处理请求，返回逻辑视图(ModelAndView)
5. DispatcherServlet → ViewResolver 解析并查询出真正的视图
6. DispatcherServlet → View 将 Model 传给 View 进行渲染
7. DispatcherServlet → 客户端，将渲染的 View 在响应中返回

### HandlerMapping 与 HandlerAdapter

HandlerMapping 与 HandlerAdapter 都是用于解决由于 Handler 具有很多类型而产生的问题

收到请求后，DispatcherServlet 通过遍历 this.handlerMappings 找到对应的 HandlerMapping，得到用于处理请求的正确类型的 Handler 并封装为 HandlerExecutionChain

得到 HandlerExecutionChain 后，DispatcherServlet 需要通过遍历 this.handlerAdapters 找到对应的 HandlerAdapter 来适配 Handler 接口与请求参数

this.handlerMappings 与 this.handlerAdapters 都由 Spirng 容器在初始化过程中读取 DispatcherServlet.properties 配置文件时注入

![](https://img-blog.csdnimg.cn/20191110200420919.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p4ZDE0MzU1MTM3NzU=,size_16,color_FFFFFF,t_70)

#### Handler(Controller) 的定义方式

实现 Controller 接口，重写 handleRequest() 方法用于请求的处理，返回 ModelAndView，Handler 最早期的实现方法

```java
import org.springframework.web.servlet.mvc.Controller;

@Component("/home")
public class HomeController implements Controller {
    
    // return null 表示不需要对响应进行视图解析
    // 直接使用 response 向客户端写回数据
    @OVerride
    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
        
        System.out.println("home");
        response.getWriter().write("home controller from body");
        return null;
        
    }
}
```

实现 HttpRequestHandler 接口，用于处理 Http Request，类似简单的 Servlet，只有 handlerRequest() 方法

```java
import org.springframework.web.HttpRequestHandler;

@Component("login")
public class LoginController implements HttpRequestHandler {
    @Override
    public void handleRequest(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("login");
        response.getWriter().write("login");
    }
}

// 对比 Servlet
@WebServlet("/myServlet")
public class MyServlet extends HttpServlet {
    
    @Override
    protected void service(HttpServletRequest req, HttpServlet resp) throws ServletException, IOException {
        super.service(req, resp);
    }
    
}
```

常用的定义方式，使用 @RequestMapping 注解：

```java
@Controller
public class IndexController {
    @RequestMapping("/index")
    @ResponseBody
    public String sayHello() {
        System.out.println("hello");
        return "hello";
    }
}
```

#### HandlerMapping

Spring 容器启动时会扫描 Handler 的实现方式，使用不同的类进行实例化，再将 URL → Handler 的映射关系存入 Map 中：

- Controller/HttpRequestHandler 接口 → BeanNameUrlHandlerMapping
- @RequestMapping → RequestMappingHandlerMapping
- 静态资源 → SimpleUrlHandlerMapping

##### 需要 HandlerMapping 实例化 Handler

处理请求时，通过循环 handlerMappings 判断请求由哪个 Handler 处理(查找 HandlerMapping)

```java
// package org.springframework.web.servlet;

protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
    if(this.handlerMappings != null) {
        for(HandlerMapping mapping : this.handlerMappings) {
            HandlerExecutionChain handler = mapping.getHandler(request);
            if(handler != null){
                return handler;
            }
        }
    }
    return null;
}
```

因为 Handler 有多种形式，处理 request 时需要通过 HandlerExecutionChain 来反射执行 Handler 中的方法，即不同的 Handler 需要 new 不同的 HandlerExecutionChain

HandlerExecutionChain 中只定义了 Object handler 属性，无法确定 Handler 类型，所有需要依赖知道类型的 HandlerMapping 进行 HandlerExecutionChain 的实例化

即：

- Handler 存在多种形式，需要 HandlerExecutionChain 通过反射执行方法
- HandlerExecutionChain 中的 handler 为 Object 属性，无法确认具体的 Handler 类型
- Spring 容器启动时会扫描 Handler 实现类型，并记录在 HandlerMapping 中
- 故需要通过 HandlerMapping 实例化 HandlerExecutionChain 来得到 Handler 类型
- HandlerMapping → HandlerExecutionChain → Handler

#### HandlerAdapter

##### HandlerAdapter 适配 Handler 与 Servlet

Handler 有很多实现形式，但 Servlet 需要处理方法的结构是固定的(以 request 和 response 作为方法入参)，故需要 ==HandlerAdapter 进行适配，模糊掉具体的实现，从而提供统一的接口(handle())给 DispatcherServlet 调用==

上述的 Handler 实现就有 3 种：

- @RequestMapping 注解：使用方法处理请求
- Controller/HttpRequestHandler  接口：使用接口实现类处理请求

##### 公共接口 vs 适配器

Spring 中采用适配器的方式，而不是通过 Handler 实现公共接口来达到统一访问接口的目的：

- 适配器无需限制 Handler 的底层实现，Handler 可以是任意类型，只需要返回其封装好的 HandlerExecutionChain 即可
- 集成第三方 Handler 时，只需要新增对应的 HandlerAdapter 即可，无需修改本地代码

### 统一异常处理

使用 @ControllerAdvice + @ExceptionHandler 注解进行统一异常处理，为所有或指定的 Controller 织入异常处理逻辑，当 Controller 中的方法抛出异常时，由被 @ExceptionHandler 注解修饰的方法处理异常

```java
@ControllerAdvice
@ResponseBody
public class GlobalExceptionHandler {

    @ExceptionHandler(BaseException.class)
    public ResponseEntity<?> handleAppException(BaseException ex, HttpServletRequest request) {
      //......
    }

    @ExceptionHandler(value = ResourceNotFoundException.class)
    public ResponseEntity<ErrorReponse> handleResourceNotFoundException(ResourceNotFoundException ex, HttpServletRequest request) {
      //......
    }
}
```

## Spring 设计模式

### IoC 与 DI

IoC 是一种设计思想，通过借助第三方来实现对象的依赖关系解耦，即程序只需要声明使用对象，而不需要关注对象的创建与管理，从而降低代码的耦合度

Spring IoC 使用类似工厂模式来实现 IoC，只需要配置文件/注解就可以令 IoC 容器负责对象的创建和管理，程序可以直接声明使用，增加项目可维护性与降低开发难度

- 控制：创建与管理对象的权力，对象 A 依赖于 对象 B，当 B 使用 A 时，需要 B 自己去创建 A
- 反转：程序 → 框架，引入 IoC 容器后，A 的创建交给容器负责，B 只需要从容器中获取 A
  - A 与 B 之间不再有依赖关系，B 获取 A 的过程由 主动创建 → 被动接收

DI 是 IoC 的一种设计模式，将实例变量注入到一个对象中

### 工厂模式

Spring 使用工厂模式通过 BeanFactory 或 ApplicationContext 创建 bean 对象

#### BeanFactory vs ApplicationContext

- 注入时机：
  - BeanFactory 使用懒加载，在启动时只创建必要的 bean，只有需要时才会注入，占用内存更少，速度更快
  - ApplicationContext 在启动时就一次性创建所有 bean
- 功能：ApplicationContext 扩展了 BeanFactory，新增了额外的功能，适用于开发环境，三种实现获取配置文件以生成 bean
  - ClassPathXmlApplication：将上下文文件作为类路径资源，从 classPath 路径下找配置文件
  - FileSystemXmlApplication：从文件系统中的 xml 文件载入上下文定义信息
  - XmlWebApplication：从 web 系统中的 xml 文件载入上下文定义信息

### 单例模式

Spring 中 bean 的默认作用域为 singleton，即 bean 全局唯一：

- 对于频繁使用的对象(重量级)，可以节省大量创建的开销
- new 操作的减少也减轻了 GC 的压力，缩短 GC 停顿时间

Spring 通过 ConcurrentHashMap 的单例注册表实现单例模式：

```java
// 通过 ConcurrentHashMap（线程安全） 实现单例注册表
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<String, Object>(64);

public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
        Assert.notNull(beanName, "'beanName' must not be null");
    	// 需要加锁防止多线程场景创建多个单例
        synchronized (this.singletonObjects) {
            // 检查缓存中是否存在实例
            Object singletonObject = this.singletonObjects.get(beanName);
            if (singletonObject == null) {
                //...省略了很多代码
                try {
                    singletonObject = singletonFactory.getObject();
                }
                //...省略了很多代码
                // 如果实例对象在不存在，我们注册到单例注册表中。
                addSingleton(beanName, singletonObject);
            }
            return (singletonObject != NULL_OBJECT ? singletonObject : null);
        }
    }
    //将对象添加到单例注册表
    protected void addSingleton(String beanName, Object singletonObject) {
            synchronized (this.singletonObjects) {
                this.singletonObjects.put(beanName, (singletonObject != null ? singletonObject : NULL_OBJECT));

            }
        }
}
```

### 代理模式

==Spring AOP 基于动态代理实现运行时增强==，当连接点实现接口时，使用 JDK Proxy 实现，否则使用 CGLIB 实现，将与业务逻辑无关，但被业务模块共享的逻辑封装，减少冗余代码，降低模块间耦合度，有利于未来的可扩展性和可维护性

AOP 也可以通过==基于字节码操作的 AspectJ 实现编译时增强==

### 模板方法

模板方法模式是一种行为设计模式，定义操作中的算法骨架，将一些实现步骤延迟到子类中，则子类的具体实现不会影响算法结构，便于算法的扩展和维护

Spring 中 JdbcTemplate、HibernateTemplate 等以 Template 结尾的对数据库操作的类即为模板方法，Spring 使用 Callback + 模板方法，达到代码复用效果，同时增强灵活性

### 观察者模式

观察者模式表示对象之间的依赖关系，当对象 A 发生改变，对象 B 也会做出反应

Spring 事件驱动模型基于观察者模式，用于解耦代码

#### 事件驱动模型

- 事件角色：ApplicationEvent 抽象类充当事件的角色
  - ContextStartedEvent：ApplicationContext 启动后触发的事件
  - ContextStoppedEvent：ApplicationContext 停止后触发的事件
  - ContextRefreshedEvent：ApplicationContext 初始化或刷新后触发的事件
  - ContextClosedEvent：ApplicationContext 关闭后触发的事件
- 事件监听者角色：ApplicationListener 接口充当事件监听者角色，定义的唯一的 onApplicationEvent() 方法处理 ApplicationEvent
- 事件发布者角色：ApplicationEventPublisher 接口充当事件的发布者，publishEvent() 方法通过 ApplicationEventMulticaster 广播事件

#### 事件流程

1. 定义事件：实现继承自 ApplicationEvent 的类
2. 定义监听者：实现 ApplicationListener 接口，重写 OnApplicationEvent() 方法
3. 定义发布者：实现 ApplicationEventPublisher 接口，重写 publishEvent() 方法

### 适配器模式

适配器令两个不兼容的接口能够一起工作

#### Spring AOP 中的适配器模式

Spring AOP 的增强通过 AdvisorAdapter 使用到适配器模式

Advice 的常用类型都有对应的拦截器，Spring 预定义的通知需要通过对应的适配器转换为 MethodInterceptor 接口类型的对象，即 Spring AOP 的方法拦截器必须实现 MethodInterceptor，适配器将未实现该接口的拦截器进行转换

如：MethodBeforeAdviceAdapter 通过 getInterceptor 方法将 MethodBeforeAdvice 适配成 MethodBeforeAdviceInterceptor

#### Spring MVC 中的适配器模式

Handler 有多种类型，而 Servlet 只能使用 request 和 response 作为参数，故需要 HandlerAdapter 作为适配器，否则就需要 DispatcherServlet 逐个判断 Handler 类型

### 装饰器模式

装饰器模式通过在原有代码上增加一层包装来进行功能扩展，比直接使用继承更加灵活，如 InputStream 通过包装成 FileInputStream 实现读取文件流

Spring 中通过装饰器模式实现在少修改原有代码下动态地切换不同的数据源，如在类名含有 Wrapper 或 Decorator 的类使用装饰器模式动态地添加额外的职责

## Spring 事务

Spring 事务能否生效取决于数据库引擎是否支持事务

MySQL 中通过 undo log 进行回滚来实现事务的原子性，所有事务进行的修改都会先记录到 undo log 中，若执行过程遇到异常，则直接利用 undo log 中的信息回滚到修改前，且 undo log 会持久化到磁盘保证不会被数据库崩溃影响

Spring 支持编程式与声明式两种事务管理方式

### 编程式事务管理

在代码中硬编码,通过 TransactionTemplate 或 TransactionManager 手动管理事务，事务范围过大时会出现事务未提交导致超时，因此事务比锁的粒度更小，适用于分布式系统

TransactionTemplate：

```java
@Autowired
private TransactionTemplate transactionTemplate;
public void testTransaction() {

        transactionTemplate.execute(new TransactionCallbackWithoutResult() {
            @Override
            protected void doInTransactionWithoutResult(TransactionStatus transactionStatus) {

                try {

                    // ....  业务代码
                } catch (Exception e){
                    //回滚
                    transactionStatus.setRollbackOnly();
                }

            }
        });
}
```

TransactionManager：

```java
@Autowired
private PlatformTransactionManager transactionManager;

public void testTransaction() {

  TransactionStatus status = transactionManager.getTransaction(new DefaultTransactionDefinition());
          try {
               // ....  业务代码
              transactionManager.commit(status);
          } catch (Exception e) {
              transactionManager.rollback(status);
          }
}
```

### 声明式事务管理

代码侵入性小，通过 AOP 实现，在 XML 配置文件中配置或直接基于注解，适用于单体应用或简单业务系统

```java
@Transactional(propagation = Propagation.REQUIRED)
public void f {
    // do someting
    // ...
}
```

### Spring 事务管理接口

- PlatformTransactionManager：平台事务管理器，事务策略的核心
- TransactionDefinition：事务定义信息(隔离级别、传播行为、超时、只读、回滚规则)
- TransactionStatus：事务运行状态

#### 事务管理器

Spring 不直接管理事务，而是提供各种事务管理器

通过 PlatformTransactionManager 为各个平台提供对应的事务管理器，由平台进行具体的实现：

- JDBC → DataSourceTransactionManager
- Hibernate → HibernateTransactionManger
- JPA → JpaTransactionManager

#### 事务属性

事务管理器接口通过 getTransaction(TransactionDefinition definition) 方法获取事务，其中需要事务属性的定义类

事务属性包含 5 个方面：

- 隔离级别
  - ISOLATION_DEFAULT：使用数据库默认的隔离级别
  - ISOLATION_READ_UNCOMMITTED：可读未提交，导致脏读、幻读、不可重读
  - ISOLATION_READ_COMMITTED：可读已提交，解决脏读，导致幻读、不可重读
  - ISOLATION_REPEATABLE_READ：可重读，解决脏读、不可重读，导致幻读
  - ISOLATION_SERIALIZABLE：将事务排序后串行处理，解决脏读、幻读、不可重读
- 超时：允许事务执行的最长时间，超时后自动回滚
- 回滚规则：定义导致回滚的异常
- 传播行为：解决业务层方法间互相调用的事务问题，当事务方法被另一个事务方法调用时，必须指定事务如何传播，如方法 A (外部方法)调用 方法 B(内部方法)，若 B 发生异常需要回滚，需要配置传播行为令 A 也回滚
  - PROPAGATION_REQUIRED：默认的传播行为，若当前存在事务，则加入，若当前不存在事务，则创建一个新事务
    - 存在被 PROPAGATION_REQUIRED 修饰的事务，则所有被 PROPAGATION_REQUIRED 修饰的内部与外部方法都属于同一事务，只要一个方法回滚，整个事务都要回滚
    - 外部方法未开启事务，则 PROPAGATION_REQUIRED 修饰的内部方法会开启自身的事务，且开启的事务相互独立
  - PROPAGATION_REQUIRES_NEW：物理外部方法是否开启事务，内部方法都开启自己独立的事务，即外部方法的回滚不会导致内部方法的回滚，但内部方法抛出满足回滚规则的异常时，外部方法也会回滚
  - PROPAGATION_NESTED：当前存在事务，则在嵌套事务内执行，当前没有事务，则同 PROPAGATION_REQUIRED，内部方法回滚不会导致外部方法回滚，外部方法回滚会导致内部方法回滚
    - 外部方法开启事务，内部方法开启新事务，作为嵌套事务
    - 外部方法无事务，内部方法单独开启一个事务
  - PROPAGATION_MANDATORY：加入当前存在的事务，若当前无事务则抛出异常
- 只读：事务只进行读操作，数据库会提供相应的优化手段
  - 在进行聚合操作时(统计查询、报表查询)，需要保证读取的数据不被修改，故需要事务支持

#### 事务状态

TransactionStatus 接口记录事务的状态，用于获取、判断事务的相应状态信息

```java
public interface TransactionStatus{
    boolean isNewTransaction(); // 是否是新的事务
    boolean hasSavepoint(); // 是否有恢复点
    void setRollbackOnly();  // 设置为只回滚
    boolean isRollbackOnly(); // 是否为只回滚
    boolean isCompleted; // 是否已完成
}
```

### @Transactional

声明式事务管理时需要使用 @Transactional 进行侵入性更小的事务管理

#### 作用域

- 方法：@Transactional 只能应用到 public 方法中，基于 Spring AOP 实现，故动态代理要求方法为 public 才能生成代理类
- 类：@Transactional 注解类表示类中的所有 public 方法都生效
- 接口

#### 配置参数

- 传播行为
- 隔离级别
- 回滚规则
- 超时
- 只读

#### 原理

@Transactional 基于 Spring AOP 实现，类/方法被标注时，Spring 容器会在启动时为其创建代理类，在调用方法时，实际调用的时 TransactionInterceptor 类中的 invoke 方法，该方法负责在目标方法执行前开启事务，执行过程中遇到异常则回滚，执行完毕后提交事务

![](https://img-blog.csdnimg.cn/2021101110475171.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZmFzdGpzb25f,size_20,color_FFFFFF,t_70,g_se,x_16)

#### 自调用问题

被 @Transactional 修饰的方法只会在被其他类调用时开启事务，同一个类方法的调用则不会生效

Spring AOP 生成的代理对象在被同一个类的方法调用时无法拦截该内部调用，也无法开启事务，因为在内部调用时，调用方会直接调用自身的方法，而不会调用代理对象的方法，若要在 Spring AOP 中使用内部调用，则需要在方法内部获取代理对象，在内部方法中调用代理对象的方法

```java
@Service
public class MyService {

private void method1() {
     ((MyService)AopContext.currentProxy()).method2(); // 先获取该类的代理对象，然后通过代理对象调用method2。
     //......
}
@Transactional
 public void method2() {
     //......
  }
}
```

故需要自调用的场景应使用 AspectJ 进行事务管理，AspectJ 会通过当前类的代理对象调用内部方法，即相当于外部调用，由此令事务注解生效

## Spring Data JPA

JPA 用于将对象数据持久化到数据库，以及从数据库中查询和恢复对象数据

### 审计功能

JPA 的审计功能用于记录数据库操作的具体行为，如某条记录的创建者、创建时间、修改者、修改时间

在字段上添加注解开启审计功能：

- @CreatedDate：表示该字段为创建时间字段，实体被 insert 时会设置值
- @CreatedBy：表示该字段为创建人字段，实体被 insert 时会设置值
- @LastModifiedDate
- @LastModifiedBy

### 实体间的关联注解

- @OneToOne
- @ManyToMany
- @OneToMany
- @ManyToOne

## Spring Security

Spring Security 基于 Spring 框架，提供了一套 Web 应用安全性的完整解决方案

### 控制请求访问权限的方法

- permitAll：允许所有访问，不做限制
- anonymous：允许匿名访问
- denyAll：无条件拒绝任何访问
- authenticated：只允许已认证的访问
- fullyAuthenticated：只允许已登录或通过 remember-me 登录的用户访问
- hasRole：只允许指定角色访问
- hasAnyRole：允许指定的任意角色访问，直需满足其中一个
- hasAuthority：只允许具有指定权限的角色访问
- hasAnyAuthority：允许拥有任意指定的角色访问，直需满足其中一个
- hasIpAddress：只允许指定 ip 访问

### hasRole vs hasAuthority

- 源码：hasRole 会自动给传入的字符串加上 ROLE_ 前缀，即数据库中的权限字符串需要加上 ROLE_ 前缀；hasAuthority 传入字符串与数据库一致即可
  - hasRole 与 hasAuthority 底层都调用 hasAnyAuthorityName
- 设计理念：用户 → 角色 → 权限，通过角色作为权限的集合可以方便地为用户分配一组权限，而不需要逐个分配

### 密码加密

Spring Security 提供多种开箱即用的加密算法，都继承自 PasswordEncoder：

```java
public interface PasswordEncoder {
    // 加密
    String encode(CharSequence var1);
    // 对比加密后的密码与原始密码
    boolean matches(CharSequence var1, String var2);
    // 判断加密密码是否需要再次加密
    default boolean upgradeEncoding(String encodedPassword) {
        return false;
    }
}
```

通过 DelegatingPasswordEncoder 兼容多种不同的密码加密算法，以代理类的方式适应不同的业务需求

## Spring Boot

Spring Boot 以约定优于配置的概念简化了 Spring 框架的配置

### Spring 框架的缺点

Spring 是重量级企业开发框架 EJB 的替代品，==通过 DI 和 AOP 用简单的 POJO 对象实现 EJB 的功能==

Spring 的组件代码是轻量级的，但它的==配置是重量级的(大量 XML 配置)==，需要将每个 bean 写入 xml 配置文件

Spring 2.5 引入了基于注解的组件扫描(@Autowired、@Resource)，消除了大量针对应用程序自身组件的显示 XML 配置，通过在相关类、方法、字段声明上使用注解，将 bean 配置移动到组件类本身，而非使用 xml 描述 bean 连接

Spring 3.0 引入基于 Java 的配置，是代替 XML 的类型安全的可重构配置方式(使用 @Bean 注入)

开启 Spring 特性时(事务管理、Spring MVC)还是需要手动进行显示配置(xml、注解)，使用第三方库时也需要在 web.xml 或 Servlet 初始化代码中等显示配置(DispatcherServlet)，且还要处理相关库的依赖和冲突

### Spring Boot 简化 Spring 开发

Spring 简化 EJB，Spring Boot 则是简化 Spring：

- 简化 Spring 应用开发，不再需要编写大量样板代码、xml 配置和注释，提高生产效率
- Spring 引导类令应用程序可以轻易地与 Spring 生态系统集成，如 Spring JDBC、Spring ORM、Spring Data、Spring Security 等
- Spring Boot 能够基于默认设置(可修改)快速地构建项目，即约定优于配置
- Spring Boot 内嵌 Http 服务器，即起步依赖中就包含 Tomcat 的 jar 包，不需要手动管理 Tomcat，可以像使用 Java 应用程序一样轻松地开发、运行、测试 web 应用程序，即不需要部署到 web 服务器也可以像 web 一样访问
- Spring Boot 提供 CLI 工具用于开发和测试 Spring Boot 应用程序
- Spring Boot 提供多种插件，可以通过内置工具(Maven、Gradle)开发和测试

### Spring Boot Starters

==Spring Boot Starters 是一系列依赖关系的集合==，用于简化依赖关系的管理

如开发 Web 应用时，需要手动添加 Spring MVC、Tomcat、Jackson 等依赖，而使用 spring-boot-starter-web 一个依赖就能包含开发 REST 服务需要的所有依赖

### 内嵌 Servlet 容器

Spring Boot 内嵌了 Tomcat、Jetty、Undertow 等，令应用程序无需部署到 web 服务器上也能像 web 一样访问

Tomcat 作为默认的嵌入式 servlet 容器，可以通过修改配置文件，移除 Tomcat 依赖并加入其他容器的依赖

### @SpringBootApplication

可以看作 @SpringBootApplication = @Configuration + @EnableAutoConfiguration + @ComponentScan：

- @Configuration：允许在上下文中注册额外的 bean (@Bean)或导入其他配置类
- @EnableAutoConfiguration：启动 SpringBoot 自动配置机制
- @ComponentScan：扫描被 @Component 注解的 bean，注解默认会扫描该类所在的包下的所有类

### Spring Boot 自动配置

==Spring Boot 基于 SPI 优化了 Spring 本身的自动装配==(@Autowired)，定义了一套接口规范：==Spring Boot 启动时会扫描外部引用 jar 包中的 META-INF/spring.factories 文件==，将文件中配置的信息加载到 Spring 容器，并执行类中定义的操作，外部的 jar 包只需要按照 Spring Boot 定义的标准即可

@EnableAutoConfiguration 是启动自动配置的关键

1. @EnableAutoConfiguration → 开启自动装配
   - @EnableAutoConfiguration 通过 Spring 的 @Import 注解导入 AutoConfigurationImportSelector 类
   - @Import 可以导入配置类或 bean 到当前类
2. AutoConfigurationImportSelector 通过 getCandidateConfiguration 方法将所有自动配置类信息以 List 形式返回，配置信息由 Spring 容器视为 bean 管理 
   - AutoConfigurationImportSelector ——实现接口→ ImportSelector ——实现方法→ selectImports ——调用方法→ getAutoConfigurationEntry 获取所有符合条件(加载到 IoC 容器)的类的全限定类名
   - ![](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/3c1200712655443ca4b38500d615bb70~tplv-k3u1fbpfcp-watermark.png)
3. getAutoConfigurationEntry()：

```java
private static final AutoConfigurationEntry EMPTY_ENTRY = new AutoConfigurationEntry();

AutoConfigurationEntry getAutoConfigurationEntry(AutoConfigurationMetadata autoConfigurationMetadata, AnnotationMetadata annotationMetadata) {
    // 检测是否开启自动装配
    if(!this.isEnabled(annotationMetadata)){
        return EMPTY_ENTRY;
    } else {
        // 获取 EncableAutoConfiguration 注解中的 exclude 和 excludeName
        AnnotationAttributes attributes = this.getAttributes(annotationMetadata);
        // 获取需要自动装配的所有配置类
        List<String> configurations = this.getCandidateConfigurations(annotationMetadata, attributes);
        // 筛选出符合条件的配置信息
        configurations = this.removeDuplicates(configurations);
        Set<String> exclusions = this.getExclusions(annotationMetadata, attributes);
        this.checkExcludedClasses(configurations, exclusions);
        configurations.removeAll(exclusions);
        configurations = this.filter(configurations, autoConfigurationMetadata);
        this.fireAutoConfigurationImportEvents(configurations, exclusions);
        return new AutoConfigurationImportSelector.AutoConfigurationEntry(configurations, exclusions);
    }
}
```

1. 检测是否开启自动装配

   ![](https://oss.javaguide.cn/p3-juejin/77aa6a3727ea4392870f5cccd09844ab~tplv-k3u1fbpfcp-watermark.png)

2. 获取 EnableAutoConfiguration 注解中的 exclude 和 excludeName

   ![](https://oss.javaguide.cn/p3-juejin/3d6ec93bbda1453aa08c52b49516c05a~tplv-k3u1fbpfcp-zoom-1.png)

3. 获取所有自动装配的配置类，读取 META-INF/spring.factories

   ```xml
   spring-boot/spring-boot-project/spring-boot-autoconfigure/src/main/resources/META-INF/spring.factories
   ```

   ![](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/58c51920efea4757aa1ec29c6d5f9e36~tplv-k3u1fbpfcp-watermark.png)

   XXXAutoConfiguration 的作用就是按需加载组件

   ![](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/94d6e1a060ac41db97043e1758789026~tplv-k3u1fbpfcp-watermark.png)

   所有 Spring Boot Starter 下的 META-INF/spring.factories 都会被读取

   故自定义的 starter 必须在 META-INF/spring.factories 中注册

   ![](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/68fa66aeee474b0385f94d23bcfe1745~tplv-k3u1fbpfcp-watermark.png)

4. 通过 @ConditionalOnXXX 过滤掉不需要加载的配置类，只有完全满足注解中条件的配置类才会被加载，加快启动速度

   - @ConditionalOnBean：容器内由指定的 bean
   - @ConditionalOnClass：当类路径下有指定类

Spring Boot 通过 @EnableAutoConfiguration 开启自动装配，通过 SpringFactoriesLoader 最终加载 META-INF/spring.factories 中的自动配置类实现自动装配，自动配置类通过 @ConditionalOnXXX 进行筛选按需加载，引入 spring-boot-starter-xxx 包实现起步依赖

#### 总结

自动装配即将第三方的 bean 对象自动装配到 Spring 容器中，而不需要手工在配置文件中写入

实现自动装配的第三方组件只需要在启动类中使用 @SpringBootApplication 注解，利用 @EnableAutoConfiguration 的三个核心功能：

- @Configuration：表示启动类为配置类，且能够引入第三方的配置类与 bean 对象，组件中也必须要有 @Configuration 配置类
- 约定第三方组件的配置文件位于其包中的 META-INF/spring.factories 文件中，通过扫描文件获取配置类在第三方 jar 包中的位置
- Spring Boot 获取所有第三方配置类后，通过 Spring 的 @Import 注解(引入 AutoConfiguartionImportSelector 类)实现对配置类的动态加载，从而完成自动装配

### 加载配置优先级

约定大于配置：在默认情况下，Spring Boot 会查找文件名为 application 的文件

1. 项目根路径的 config 文件夹
2. 项目根路径
3. classpath 下的 config 文件夹
4. classpath

![](https://img-blog.csdnimg.cn/20210123234010792.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0NoYWt5cmQ=,size_16,color_FFFFFF,t_70)

优先级高的配置会覆盖优先级低的

当存在同级配置文件时：.properties > .yml > .yaml

#### 特殊配置

- 指定读取的子文件

![](https://img-blog.csdnimg.cn/20210127171439313.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0NoYWt5cmQ=,size_16,color_FFFFFF,t_70)

- 命令行指定读取的配置文件时，只会读取指定的配置文件，不再读取任何其他配置文件

![](https://img-blog.csdnimg.cn/20210129170300609.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0NoYWt5cmQ=,size_16,color_FFFFFF,t_70)

### Spring Boot 启动流程

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6ba8bf5c8177430b8f462f35948d1c74~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

1. Java 程序由启动主类调用 main() 开始
2. 调用 SpringApplication 的构造方法，实例一个 Spring 应用对象，在构造方法完成启动环境初始化工作：推断主类、Spring 应用类型、加载配置文件、读取 spring.factories 文件等
3. 调用 SpringApplication.run 方法，所有启动工作在该方法中完成：加载配置资源、上下文的准备/创建/刷新、过程时间发布等

#### 启动入口

启动类 = @SpringBootApplication + psvm 方法 + SpringApplication.run(XXXApplication.class, args)

- @SpringBootApplication：
  - @Configuration：标记启动类为 IoC 容器的配置类，同时也能够引入第三方的配置类与 bean 对象
  - @EnableAutoConfiguration：自动装配的核心，通过 Spring 的 ==@Import== 注解引入 ==AutoConfigurationImportSelector== 类，扫描约定的第三方配置文件路径(==jar 包的 META-INF/spring.factories==)以获取第三方配置类(==getAutoConfigurationEntry==)，筛选(==@ConditionalOnXXX==)并加载(==SpringFactoryLoader==)需要的配置类
  - @ComponentScan：指定扫描 bean 对象的路径，默认为启动类所在包下的所有类，将 Spring 项目自身的 bean 对象加载至 IoC 容器

#### SpringApplication 构造器

```java
public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
    this.resourceLoader = resourceLoader;
    Assert.notNull(primarySources, "PrimarySources must not be null");
    this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
    // 判断当前程序类型
    this.webApplicationType = WebApplicationType.deduceFromClasspath();
    // 使用 SpringFactoriesLoader 实例化所有可用的初始器
    setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));
    // 使用 SpringFactoriesLoader 实例化所有可用的监听器
    setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
    // 配置应用主方法所在类
    this.mainApplicationClass = deduceMainApplicationClass();
}
```

- 判断当前程序类型：通过 classpath 中是否存在特征类(ConfigurableWebApplicationContext)决定是否应该创建一个为 Web 应用使用的 ApplicationContext 类型(NONE/SERVLET/REACTIVE)

#### 启动方法 run()

初始化后由 run 方法完成 Spring 的整个启动过程：

- 准备 Environment
- 发布事件
- 创建上下文、bean
- 刷新上下文
- 结束

```java
public ConfigurableApplicationContext run(String... args) {
    // 开启时钟计时
    StopWatch stopWatch = new StopWatch();
    stopWatch.start();
    // Spring 上下文，注入与管理 bean 且提供多种高级功能
    ConfigurableApplicationContext context = null;
    // 启动异常报告容器
    Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
    // 开启设置，让系统模拟不存在 IO 设备
    configureHeadlessProperty();
    // 初始化 SpringApplicationRUnListener 监听器，并进行封装
    SpringApplicationRunListeners listeners = getRunListeners(args);
    listeners.starting();
    try {
        ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
        // 准备 Environment
        ConfigurableEnvironment environment = prepareEnvironment(listeners, applicationArguments);
        configureIgnoreBeanInfo(environment);
        Banner printedBanner = printBanner(environment);
        // 实例化上下文
        context = createApplicationContext();
        // 异常播报器
        exceptionReporters = getSpringFactoriesInstances(SpringBootExceptionReporter.class, new Class[] {ConfigurableApplication.class}, context);
        // 初始化容器
        prepareContext(context, environment, listeners, applicationArguments, printedBanner);
        // 刷新上下文
        refreshContext(context);
        // 给实现类的 hook
        afterRefresh(context, applicationArguments);
        stopWatch.stop();
        if(this.logStartupInfo) {
            new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), stopWatch);
        }
        listeners.started(context);
        callRunners(context, applicationArguments);
    }
    catch (Throwable ex) {
        handleRunFailure(context, ex, exceptionReporters, listeners);
        throw new IllegalStateException(ex);
    }
    try {
        lsiteners.running(context);
    }
    catch(Throwable ex) {
        handleRunFailure(context, ex, exceptionReporters, null);
        throw new IllegalStateException(ex);
    }
    return context;
}
```

- SpringApplicationRunListener：作为 Spring Boot 的监听器，在整个启动流程接收不同执行点事件通知，并广播相应的事件，用户可以通过监听器接口回调在启动的各个流程加入自定义逻辑
  - starting：run 方法开始执行前
  - environmentPrepared：在 environment 准备完成后，context 创建前
  - contextPrepared：在 context 构建完成时
  - contextLoaded：在 context 构建完成，还未刷新时
  - started：在 context 刷新且启动后
  - running：在 run 方法执行完成前
  - failed：应用运行出现错误时

```java
public interface SpringApplicationRunListener {
  /**EventPublishingRunListener 前期采用 SimpleApplicationEventMulticaster.multicastEvent(ApplicationEvent) 进行广播
  **/
   default void starting() {} 
   default void environmentPrepared(ConfigurableEnvironment environment) {}
   default void contextPrepared(ConfigurableApplicationContext context) {}
   default void contextLoaded(ConfigurableApplicationContext context) {}
  /**
  EventPublishingRunListener 后期采用 context.publishEvent(ApplicationEvent)
  **/
   default void started(ConfigurableApplicationContext context) {}
   default void running(ConfigurableApplicationContext context) {}
   default void failed(ConfigurableApplicationContext context, Throwable exception) {}
}

```

- prepareEnvironment：运行时使用的 ==Environment 只读接口==是对运行程序的抽象，保存系统配置的中心，启动时使用的是==可编辑的 ConfigurableEnvironment 接口==，提供合并父环境、添加 active profile(切换 dev/prd)以及设置解析配置文件方式的接口
  - MutablePropertySources getPropertySources()：返回可编辑的 PropertySources，用于在启动阶段自定义环境的 PropertySources

```java
private ConfigurableEnvironment prepareEnvironment(SpringApplicationRunListeners listeners,
      ApplicationArguments applicationArguments) {
   // Create and configure the environment
  //根据不同环境不同的Enviroment （StandardServletEnvironment，StandardReactiveWebEnvironment，StandardEnvironment）
   ConfigurableEnvironment environment = getOrCreateEnvironment();
  //填充启动类参数到enviroment 对象
   configureEnvironment(environment, applicationArguments.getSourceArgs());
  //更新参数
  ConfigurationPropertySources.attach(environment);
  //发布事件 
  listeners.environmentPrepared(environment);
  //绑定主类 
  bindToSpringApplication(environment);
   if (!this.isCustomEnvironment) {//转换environment的类型，但这里应该类型和deduce的相同不用转换
      environment = new EnvironmentConverter(getClassLoader()).convertEnvironmentIfNecessary(environment,
            deduceEnvironmentClass());
   }
  //将现有参数有封装成proertySources
   ConfigurationPropertySources.attach(environment);
   return environment;
}

```



### 常用注解

- 自动装配：

  - SpringBootAppliaction：@Configuration + @EnableAutoConfiguration + @ComponentScan，自动装配的基础
  - @Configuration：允许引入其他配置类或 bean
  - @EnableAutoConfiguration：启用自动装配
  - @ComponentScan：扫描 @Component 的 bean，默认扫描该注解所在包下的所有类

- Spring Bean：

  - @Autowired：自动注入对象(@Service → @Controller)，被注入的类同样要被 Spring 容器管理，默认 byType，失效时 byName
  - @Component：通用的类注解，可标注任意类为 Spring 组件

  - @Repository：对应持久层(DAO)，主要用于数据库相关操作，会将数据库返回的异常转换为 Java 的异常抛出
  - @Service：对应服务层，只起标注作用
  - @Controller：对应控制层，用于接收用户请求并返回，负责请求的路由、转发、处理、响应
  - @RestController：@Controller + @ResponseBody，表示类为控制器 bean，且是==将函数的返回值直接填入 HTTP Response Body 中==，是 REST 风格的控制器
    - 使用 @Controller 时，需要返回一个 View，用于前后端一体的传统 Spring MVC 架构
  - @Scpoe：声明 bean 的作用域
    - singleton
    - prototype
    - session
    - application
    - request
  - @Configuration：标识配置类，只有在 @Configuration 标注下的 @Bean 才为单例模式

- HTTP 请求：

  - @RequestParam：获取查询参数

  - @Pathvariable：获取路由参数

  - @RequestBody：读取 Request 请求的 body 部分且 Content-Type = application/json，接收到数据后会自动绑定到 Java 对象
    - 通过 HttpMessageConverter 负责 Json → Java 对象
    - @RequestBody 唯一，但 @RequestParam 与 @Pathvariable 能有多个

- 读取配置信息：

  - @Value("${}")：读取简单的配置信息
    - 不能作用于 static/final
    - 不能在非注册的类中使用，即未被 @Component 等修饰
    - 被 @Value 修饰后只能通过依赖注入的方式使用，不能通过 new 的方式自动注入
  - @ConfigurationProperties：读取配置信息并与 bean 绑定
  - @PropertySource：读取指定 properties 文件，不支持 yml

- 参数校验：

  - @NotEmpty
  - @NotBlank
  - @Null/@NotNull
  - @AssertTrue/@AssertFalse
  - @Pattern(regex=, flag=)
  - @Email
  - @Min(value)/@Max(value)/@DecimalMax(value)/@DecimalMin(value)
  - @Size(max=, min=)
  - @Digits(integer, fraction)
  - @Past/@Future
  - @Valid：在需要验证的参数上添加，验证失败则抛出错误
  - @Validated：在类上添加，告知 Spring 校验方法参数
    - 校验 Service 层时需要，Controller 层时不需要

- 异常处理：

  - @ControllerAdvice：注解定义全局异常处理类
  - @ExceptionHandler(Exception.class)：注解声明异常处理方法

- JPA：

  - @Entity：声明类对应数据库实体
  - @Table：设置表名
  - @Id：声明主键
    - @GeneratedValue 指定主键生成策略：
      - TABLE：使用指定的表保存主键
      - SEQUENCE：在不支持主键自增的数据库中，使用序列机制生成主键
      - IDENTITY：自增
      - AUTO：由数据库引擎决定
  - @Column：声明字段
  - @Transient：声明不需要持久化的字段
  - @Lob：声明大字段
  - @Enumeration：声明枚举类型字段
  - @EnableJpaAuditing：开启审计功能，需要继承 AbstractAuditBase
  - @Modifying：配合 @Transactional，标识该操作为修改操作
  - @OneToOne/@OneToMore/@MoreToMore/@MoreToOne

- 事务：

  - @Transactional
    - 作用于类时，会作用于类中所有 public 方法
    - 作用于方法时，会覆盖类的 @Transactional

- Json：

  - @JsonIgnoreProperties：作用于类，过滤不返回/不解析的字段
  - @JsonIgnore：作用于字段
  - @JsonFormat：格式化 Json，扁平化对象

- 测试：

  - @ActiveProfiles：作用于测试类，声明生效的 Spring 配置文件
  - @Test：声明测试方法
  - @Transactional：回滚测试数据
  - @WithMockUser：Spring Security 中用于模拟真实用户，能够赋予权限

- 定时任务：

  - @Scheduled
  - @EnableScheduled：在启动类标注后，Spring Boot 会发现 @Scheduled 并在后台执行任务

### Spring Boot 监控运行状态

使用 Spring Boot Actuator 对 Spring Boot 进行简单的监控，集成模块后能够使用开箱即用的 API 获取程序运行时的内部状态信息

如：通过 GET 方法访问 /health 接口获取应用程序的健康指标

