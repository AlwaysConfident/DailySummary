# Spring

轻量级的开源框架，用于提高开发效率与系统的可维护性

Spring Framework 可以方便地集成第三方模块，无需重新造轮子，开箱即用

## 主要模块

![Spring 各个模块的依赖关系](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/20200902100038.png)

- Core Container：核心模块，提供 IoC 依赖注入功能，其他功能基本都需要依赖该模块
- AOP：提供面向切面编程的功能
- Data Access/Integration：提供对数据库访问的抽象 JDBC(适配器模式)，Java 只需要与 JDBC 交互，而不需要关注底层的数据库实现
- Web：提供 Web 相关功能的实现(MVC、Socket)
- Messaging：Spring 4.0 引入，负责为 Spring 框架集成基础报文传送应用
- Spring Test：Spring 团队提倡测试驱动开发(TDD)，通过 IoC 简化了单元测试与集成测试，Spring Test 模块对主流测试框架(JUnit)都有较好的支持

### Spring vs Spring MVC vs Spring Boot

- Spring 即 Spring Framework，提供了核心的 Core Container 模块，其他功能大多需要依赖该模块提供的 IoC 功能
- Spring MVC 为 Spring 中的一个重要模块，主要用于快速构建 MVC 架构的 Web 程序
- Spring Boot 简化了 Spring 开发所需的配置(需要使用 XML/Java 对用到的功能显式配置)，实际的开发仍由其他模块实现(Spring MVC 构建 MVC 架构)

## Spring IoC