# Java I/O

I/O 流即数据输入输出流，从外部输入到内存为输入流，从内存输出到外部为输出流，根据流传输内容处理方式又分为字符/字节 I/O 流

- InputStream/OutputStream：字节
- Reader/Writer：字符

Java I/O 流是有序(FIFO，但 RandomAccessFile 除外)，只读/只写(输入流只能读，输出流只能写)的

## 字节流

- InputStream：
  - FileInputStream：读取文件字节流，可指定文件路径，可直接读取单字节数据，也可读取至字节数组中
  - DataInputStream：用于读取指定类型数据，不可单独使用，必须结合其他流
  - ObjectInputStream：用于从输入流中读取 Java 对象(反序列化)
- OutputStream：
  - FileOutputStream
  - DataOutputStream
  - ObjectOutputStream

## 字符流

- 字节流由 JVM 将字节转换而成，这个过程需要耗时
- 字节流可能存在编码问题

音频、图片等媒体文件使用字节流，字符使用字符流操作(字符流只能处理纯文本)

- Reader：
  - InputStreamReader：字节流转换为字符流的桥梁
    - FileReader：封装 InputStreamReader，可以直接操作字符文件
- Writer：
  - OutputStreamWriter
    - FileWriter

## 字节缓冲流

I/O 操作消耗性能高，缓冲流将数据加载至缓冲区，一次性读取/写入多个字节，从而避免频繁的 I/O 操作

字节缓冲流通过装饰器模式增强字节流类对象

```java
BufferedInputStream bufferedInputStream = new BufferedInputStream(new FileInputStream("input.txt"));
```

字节缓冲流通过内部的缓冲区(字节数组)先将读取到的字节存放在缓冲区

## 字符缓冲流

内部维护一个字节数组作为缓冲区

## 随机访问流

RandomAccessFile，支持从指定位置开始读写文件，其实现依赖于 FileDescriptor(文件描述符)与 FileChannel(内存映射文件)

- r：只读模式
- w：只写模式
- rw：读写模式
- rws：读写，对文件内容或其元数据的修改同步更新到外部存储设备
- rwd：读写，对文件内容的修改同步到外部存储设备

通过 seek() 设置偏移量来实现到达指定位置

可以用于断点续传，将大文件分片，文件传输中断后，只需要上传未完成的分片即可，RandomAccessFile 可以合并分片

## I/O 设计模式

### 装饰器模式

装饰器模式通过组合代替继承的方式为原类增加新的功能，一个装饰器可以增强多个不同的类，而不需要像继承一样为它们都新增一个子类，且装饰器可以方便地嵌套使用

如：BufferedInputStream 可以装饰任意 InputStream 的子类

```java
BufferedInputStream bufferedInputStream = new BufferedInputStream(new FileInputStream("input.txt"));
```

装饰器类需要与原始类继承相同的抽象类或实现相同的接口

### 适配器模式

类比电源适配器，适配器模式将被调用方进行包装，使其能够被调用方使用，从而协调互不兼容的两个接口

如：InputStreamReader 使用 StreamDecoder 对字节码进行解码，实现字节流到字符流的转换

```java
InputStreamReader isr = new InputStreamReader(new FileInputStream(fileName), "UTF-8");
BufferedReader bufferedReader = new BufferedReader(isr);
```

#### 装饰器模式 vs 适配器模式

- 装饰器模式着重于增强原类的方法，令装饰后的类具有新的方法
- 适配器模式注重于兼容两个不匹配的接口，令适配后的类/方法能够被调用方使用

## 工厂模式

工厂模式用于创建对象，NIO 中大量运用工厂模式

如：Files 类的 newInputStream 方法用于创建 InputStream 对象

## 观察者模式

订阅-观察模式，被观察的对象发生变化时，所有注册的订阅者都会收到通知(触发监听事件)

NIO 中的文件目录监听服务基于 WatchService(观察者) 接口和 Watchable(被观察者) 接口

Watchable 接口定义了一个用于将对象注册到 WatchService 并绑定监听事件的方法 register

```java
public interface Path extends Comparable<Path>, Iterable<Path>, Watchable {
    
    public interface Watchable {
        WatchKey register(WatchService watcher, WatchEvent.Kind<?>[] events, WatchEvent.Modifier... modifiers) throws IOException;
    }
    
}
```

WatchService 用于监听文件目录的变化，同一个  WatchService 对象能够监听多个文件目录

```java
// 创建 WatchService 对象
WatchService watchService = FileSystems.getDefault().newWatchService();

// 初始化一个被监控文件夹的 Path 类
Path path = Paths.get("directory");
// 将 Path 注册到 WatchService 对象中
WatchKey watchKey = path.register(watchService, StandardWatchEventKinds...);
```

监听事件：

- StandardWatchEventKinds.ENTRY_CREATE：文件创建
- StandardWatchEventKinds.ENTRY_DELETE：文件删除
- StandardWatchEventKinds.ENTRY_MODIFY：文件修改

通过 register 方法返回的 WatchKey 对象可以获取事件的具体信息

WatchService 内部通过一个 daemon thread(守护线程)采用定期轮询的方式检测文件的变化

## I/O 模型

操作系统中，只有内核态才能进行系统级别的资源操作(文件管理，进程通信，内存管理等)，故应用程序进行 I/O 操作只能发起系统调用，等待内核处理，即应用程序 I/O 的流程为：

1. 应用程序发出系统调用，等待内核处理
2. 内核等待 I/O 设备准备好数据
3. 内核将数据从内核空间拷贝到用户空间

### BIO(Blocking I/O)

同步阻塞 I/O 模型

应用程序发起 read 调用后，会一直阻塞等待内核将数据拷贝到用户空间

![图源：《深入拆解Tomcat & Jetty》](https://oss.javaguide.cn/p3-juejin/6a9e704af49b4380bb686f0c96d33b81~tplv-k3u1fbpfcp-watermark.png)

在高并发时性能无法接受

### NIO(Non-blocking/New I/O)

Java 4 中引入，支持面向缓冲，基于通道的 I/O 操作方法，对于高负载、高并发的应用，应使用 NIO(在连接数较少，并发程度低，网络传输速度快的场景，NIO 的性能不一定优于 BIO)

#### 同步非阻塞 I/O 模型

![图源：《深入拆解Tomcat & Jetty》](https://oss.javaguide.cn/p3-juejin/bb174e22dbe04bb79fe3fc126aed0c61~tplv-k3u1fbpfcp-watermark.png)

应用程序会一直发起 read 调用，等待内核将数据拷贝到用户空间，在发起调用的期间是阻塞的，但通过轮询操作避免了一直阻塞

由于应用程序需要不断发起 read 调用，会产生大量 CPU 开销

#### 多路复用 I/O 模型

![](https://oss.javaguide.cn/github/javaguide/java/io/88ff862764024c3b8567367df11df6ab~tplv-k3u1fbpfcp-watermark.png)

I/O 多路复用模型中，通过 select/epoll(select 调用的增强版本) 调用询问内核数据是否已准备好，在内核拷贝完成后才真正进行 read 调用(将数据从内核空间 -> 用户空间)并阻塞线程

I/O 多路复用模型通过减少无效的系统调用，减少了对 CPU 资源的消耗

NIO 中使用 selector(选择器/多路复用器)来管理多个客户端连接，只有当客户端数据到达时才会为其服务，而不是一直阻塞等待

![Buffer、Channel和Selector三者之间的关系](https://oss.javaguide.cn/github/javaguide/java/nio/channel-buffer-selector.png)

#### NIO 核心组件

- Buffer：数据的缓冲区，Channel 从 Buffer 中读写数据
- Channel：数据输入输出的双向传输通道，可以是文件/socket/数据源之间的连接
- Selector：基于事件驱动的选择器，任何 Channel 都可以注册到 Selector 中，由 Selector 来分配线程处理事件
  - Selector 不断轮询注册的 Channel，当一个 Channel 准备好时会处于就绪状态，此时 Selector 会将其加入到就绪集合中。通过 SelectorKey 可以获取就绪集合，然后对这些就绪的 Channel 进行响应的 I/O 操作

#### 零拷贝

NIO 通过零拷贝减少数据在存储区域之间拷贝的次数，以此来减少上下文切换和 CPU 的拷贝时间

Java 对零拷贝的支持：

- MappedByteBuffer：基于 Linux 内核的 mmap 系统调用，**将文件映射到系统内存中**，形成一个虚拟内存文件，由此来直接操作内存中的数据而不需要通过系统调用读写文件
- FileChannel：基于 Linux 内核的 sendfile 系统调用，在网络传输中**直接将文件数据从磁盘发送到网络**，而不需要经过用户空间的缓冲区

### AI/O(Asynchronous I/O)

AIO 是 Java 7 中引入的异步 I/O，基于事件和回调机制实现

应用程序发起 read 程序后异步进行其他操作，内核完成拷贝后操作系统调用回调函数告知相应的线程数据已准备好

![](https://oss.javaguide.cn/github/javaguide/java/io/3077e72a1af049559e81d18205b56fd7~tplv-k3u1fbpfcp-watermark.png)

### BI/O vs NI/O vs AI/O

![BIO、NIO 和 AIO 对比](https://oss.javaguide.cn/github/javaguide/java/nio/bio-aio-nio.png)