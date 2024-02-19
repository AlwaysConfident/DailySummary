# Unsafe

Unsafe 是位于 sun.misc 包下的类，主要提供一些用于执行底层，不安全的操作

- 底层：直接访问系统内存资源，自主管理内存资源等，这些方法能够提示 Java 运行效率，增强 Java 语言底层资源操作能力
- 不安全：由于 Unsafe 令 Java 拥有了类似 C 指针的操作内存空间的能力，所以也会造成相关的指针问题，故 Java 变得不安全

Unsafe 提供的功能的实现依赖于由 native 修饰的本地方法，即由其他编程语言(能够直接操作内存空间的语言)编写，Java 代码中只是声明方法头，具体的实现则交给本地代码

使用本地方法的好处：

- Java 中不具备依赖于操作系统的特性，故 Java 在实现跨平台的同时要实现对底层的控制只能借助其他语言实现
- 有些功能在其他语言中已有比较完备的实现，Java 可以直接调用
- 程序对时间敏感或对性能要求极高时，需要使用更加底层的语言

## Unsafe 创建

```java
public final class Unsafe {
  // 单例对象
  private static final Unsafe theUnsafe;
  ......
  private Unsafe() {
  }
  @CallerSensitive
  public static Unsafe getUnsafe() {
    Class var0 = Reflection.getCallerClass();
    // 仅在引导类加载器`BootstrapClassLoader`加载时才合法
    if(!VM.isSystemDomainLoader(var0.getClassLoader())) {
      throw new SecurityException("Unsafe");
    } else {
      return theUnsafe;
    }
  }
}
```

Unsafe 类使用单例模式，且只有启动类加载器加载的类才能调用 Unsafe 类中的方法，来防止不可信代码的调用

### 获取实例

- 利用反射机制(反射可以无视访问限制修饰符)获取 Unsafe 类中已经实例化的单例对象 theUnsafe

  ```java
  private static Unsafe reflectGetUnsafe() {
      try {
        Field field = Unsafe.class.getDeclaredField("theUnsafe");
        field.setAccessible(true);
        return (Unsafe) field.get(null);
      } catch (Exception e) {
        log.error(e.getMessage(), e);
        return null;
      }
  }
  ```

- 通过 Java 命令行 -Xbootclasspath/a 把调用 Unsafe 相关方法的类 A 所在的 jar 包路径追加到默认的 bootstrap 路径中，使得 A 被引导类加载器加载

  ```java
  java -Xbootclasspath/a: ${path}   // 其中path为调用Unsafe相关方法的类所在jar包路径
  ```

### Unsafe 功能

- 内存操作：内存的分配/调整/拷贝/清除，由于此处操作的内存为堆外内存，所以不会参与 GC，需要手动回收(freeMemory)
  - 堆外内存：即 JVM 堆外，由操作系统直接管理的内存空间。使用堆外内存能减小堆内内存的规模，从而减小 GC 时停顿对应用的影响；同时，由于堆外内存不需要经过 JVM 管理，即其 I/O 直接与操作系统交互，效率更高
- 内存屏障：编译器和 CPU 会在保证程序输出结果一致的情况下，对代码进行重新排序，从指令优化角度提高性能(避免一直等待一个指令执行)。而指令重排序会导致 CPU 的高速缓存和内存中的数据不一致，内存屏障即是通过组织屏障两边的指令重新排序从而避免编译器和硬件的不正确优化情况
  - 屏障也可看作同步点，即在此之前的所有读写操作都执行完后才能开始执行其后的操作
- 对象操作：对象成员属性的内存偏移量获取，以及字段属性值的修改，对象的实例化
  - 只需要 Class 对象就可以实例化对象，不需要调用构造函数、初始化代码、JVM 安全检查等
- 数组操作：定位数组中每个元素在内存中的位置
- CAS 操作：Compare And Swap，是实现并发算法时常用的一种技术。CAS 操作包含三个操作数——内存位置、预期原值及新值。执行 CAS 操作时，将内存位置的值与预期原值比较，匹配则处理器会自动将该位置值更新为新值，否则不做操作
  - CAS 为原子操作，不会造成数据不一致的问题
- 线程调度：对线程的挂起与恢复
- Class 操作：类加载与静态变量的操作
- 系统信息：获取系统信息