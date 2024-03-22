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

