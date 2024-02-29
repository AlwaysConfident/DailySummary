# Java 8 新特性

## Interface

- default：默认方法，可以被实现类继承，避免新增接口方法需要修改所有实现类
- static：接口的静态方法，只能由接口类调用，不能被实现类重写(静态绑定，静态方法的调用在编译期确定)

### interface vs abstract class

- 修饰符：interface 中的方法只能是 public abstract，属性只能是 public static final；abstract class 则可以使用各种修饰符
- 多继承：实现类可以实现多个 interface；抽象类只能用于单继承
- 实现思想：interface 的思想是作为一种方法的扩展；abstract class 的思想是利用继承

## functional interface 函数式接口

Single Abstract Method Interface(SAM)，有且只有一个抽象方法，但可以有多个非抽象方法的接口

Java 8 中的 java.util.function 包下的所有接口都有 @FunctionalInterface 注解，提供函数式编程

其他包中没有 @FunctionalInterface 注解，但只要符合函数式接口定义就是函数式接口，与注解无关，注解只在编译时期强制规范定义的作用

### Lambda 表达式

Lambda 表达式是一个匿名函数，Java 8 允许把函数作为参数传递到方法中

- 替代内部类实现动态参数：只要方法的参数是函数式接口，都可以用 Lambda 表达式

  ```java
  // 内部类
  new Thread(new Runnable() {
      @Override
      public void run() {
          System.out.println("test");
      }
  }).start();
  
  // Lambda 
  new Thread(() -> System.out.println("test")).start();
  ```

- 集合迭代

- 方法引用：使用 `::` 关键字传递方法或构造函数引用，表达式返回的类型必须是 functional-interface

  ```java
  public class LambdaClassSuper {
      LambdaInterface sf(){
          return null;
      }
  }
  
  public class LambdaClass extends LambdaClassSuper {
      public static LambdaInterface staticF() {
          return null;
      }
  
      public LambdaInterface f() {
          return null;
      }
  
      void show() {
          //1.调用静态函数，返回类型必须是functional-interface
          LambdaInterface t = LambdaClass::staticF;
  
          //2.实例方法调用
          LambdaClass lambdaClass = new LambdaClass();
          LambdaInterface lambdaInterface = lambdaClass::f;
  
          //3.超类上的方法调用
          LambdaInterface superf = super::sf;
  
          //4. 构造方法调用
          LambdaInterface tt = LambdaClassSuper::new;
      }
  }
  ```

- 访问变量：lambda 表达式可以引用外部变量，但该变量默认为 final，无法被修改

  ```java
  int i = 0;
  Collections.sort(strings, (Integer o1, Integer o2) -> o1 - i);
  ```

## Stream

Stream 不存储数据，可以检索和逻辑处理集合数据(筛选、排序、统计、计数)，它的源数据可以是 Collection、Array，由于它的方法参数都是函数式接口，所以一般和 Lambda 配合使用

- stream 串行流
- parallelStream 并行流，可多线程执行

### 延迟执行

返回 Stream 对象的方法会在使用时才执行

```java
@Test
public void laziness(){
  List<String> strings = Arrays.asList("abc", "def", "gkh", "abc");
  Stream<Integer> stream = strings.stream().filter(new Predicate() {
      @Override
      public boolean test(Object o) {
        System.out.println("Predicate.test 执行");
        return true;
        }
      });

   System.out.println("count 执行");
   stream.count();
}
/*-------执行结果--------*/
count 执行
Predicate.test 执行
Predicate.test 执行
Predicate.test 执行
Predicate.test 执行
```

- Stream 通过简单的链式编程，使得它可以方便地对遍历处理后的数据进行再处理
- 方法参数都是函数式接口
- 一个 Stream 只能操作一次，操作完成后就关闭
- Stream 不保存也不修改数据源

### map vs flatMap

map 与 flatMap 都将一个函数应用于集合中的每个元素，但 map 返回一个新的集合，flatMap 将每个元素都映射为一个集合，最后将集合展平

## Optional

Optional 用于解决 NPE 问题，其中可以包含空值或非空值

## Date-Time API

java.util.Date 的问题：

- 非线程安全
- 时区处理麻烦
- 格式化、时间计算繁琐
- 设计缺陷，Date 类同时包含日期和时间

java.time 是对 Date 类的补充增强：

- 日期与时间的分离
- 格式化
- 字符串转日期
- 日期计算
- 获取指定日期
- JDBC 与 Java 8
- 时区转换