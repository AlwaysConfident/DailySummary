# Netty

Netty 是<mark>基于 NIO 的 cs 框架</mark>，可以快速简单地开发网络应用程序，简化并优化了 TCP 和 UDP 套接字服务器等网络编程，提升了性能与安全性，<mark>支持多种协议</mark>(FTP/SMTP/HTTP)

Netty 主要用于网络通信：

- RPC 框架的网络通信工具
- HTTP 服务器
- 即时通讯系统
- 消息推送系统

## NIO

同步非阻塞的 I/O 模型，面向缓存、基于通道的 I/O 操作方法

- 阻塞：工作线程向内核发起 read 调用后等待内核将数据拷贝到用户空间，该过程中无法接收其他请求，对于客户端来说应用被阻塞
- 非阻塞：使用专门的工作线程向内核发起 read 调用后，该工作线程被阻塞，但请求线程仍能接收其他请求，对于该请求来说工作是同步的

直接使用 JDK NIO 的编程模型复杂，难以处理断连重连、丢包、粘包等问题

使用 Netty 能够简单地实现

## 核心组件

### ByteBuf

用于处理字节流的字节容器，底层为字节数组

`ByteBuf`维护了两个不同的索引：一个用于读取，一个用于写入。当你从`ByteBuf`读取时，它的`readerIndex`将会被递增已经被读取的字节数。同样地，当你写入`ByteBuf`时，它的`writerIndex`也会被递增。

#### 使用模式

- 堆缓冲区：将 ByteBuf 存储在 JVM 的堆中，能在没有使用池化的情况下提供快速的分配和释放。

- 直接缓冲区：堆外缓冲，可以避免通过 socket 发送数据前从堆缓冲区复制到直接缓存区的开销
  
  - 因为不受 JVM 的垃圾回收管理，直接缓冲区的空间分配和释放更加昂贵，且处理数据时需要将其复制到 JVM 中

- 复合缓冲区：为多个`ByteBuf`提供一个聚合视图。

#### 字节级操作

- 随机访问索引：类似数组下标，使用那些需要一个索引值参数的方法（的其中）之一来访问数据既不会改变`readerIndex`也不会改变`writerIndex`。

- 顺序访问索引：ByteBuf 被读索引和写索引分为三个区域
  
  - 可丢弃字节：已经被读过，可以丢弃的字节，调用 discardReadBytes 方法可以领可丢弃字节分段变为可写(仅仅移动写索引，并不会擦除原本的内容)，且为了将可读字节移至缓冲区开始，可能会导致内存复制
  
  - 可读字节：可读字节分段存储了实际数据。
  
  - 可写字节：一个拥有未定义内容的、写入就绪的内存区域。

#### 分配

- ByteBufAllocator：为了降低分配和释放内存的开销，Netty通过`interface ByteBufAllocator`实现了（`ByteBuf`的）池化，它可以用来分配我们所描述过的任意类型的`ByteBuf`实例。可以通过`Channel`（每个都可以有一个不同的`ByteBufAllocator`实例）或者绑定到`ChannelHandler`的`ChannelHandlerContext`获取一个到`ByteBufAllocator`的引用。

- Unpooled 缓冲区：提供了静态的辅助方法来创建未池化的`ByteBuf`实例。

- ByteBufUtil：提供了用于操作`ByteBuf`的静态的辅助方法。

### Bootstrap/ServerBootstarp

客户端与服务端的启动引导类

bind() → 绑定本地端口，客户端用于 UDP，服务端用于接收客户端连接

connect() → 客户端连接服务器的指定端口

客户端只需要一个 EventLoopGroup 用于处理与服务端的连接

服务端需要两个 EventLoopGroup，一个用于监听来自客户端的连接请求，连接建立后会构造新的 Channel 再分配给另一个 EventLoopGroup 处理

### Channel

网络操作抽象类，客户端与服务器建立连接后通过 Channel 进行网络 I/O

它代表一个到实体（如一个硬件设备、一个文件、一个网络套接字或者一个能够执行一个或者多个不同的I/O操作的程序组件）的开放连接，如读操作和写操作

### EventLoop

Netty 最核心的概念

负责监听网络事件并调用事件处理器进行相关 I/O 操作处理

Channel 为 Netty 的网络操作抽象类，EventLoop 负责处理注册到其上的 Channel 的 I/O 操作(包括监听到的 Channel 的 IO 事件与 ChannelHandler 的业务逻辑)

EventLoopGroup 包含多个 EventLoop 且管理它们生命周期

EventLoop 通常内部包含一个线程，处理的 I/O 事件都将在其内部的线程中，保证线程安全

对于定时任务，EventLoop 会严格按照执行队列中的任务顺序执行(即在到达指定时间后也不会中断当前任务)，对于过期的定期任务，根据不同的调度策略会有不同的执行方法：

- 固定时间：只有在执行定期任务时才会计算下一次的执行时间，例如对于周期为 10s 的定期任务，延迟至 25s 才执行，则下次执行会在 35s

- 固定速率：当检测到任务延期时，会尽量补偿错过的任务，例如 25s 时会执行两次定期任务，且下一次执行在 30s

#### 线程管理

Netty 会通过调用`EventLoop`的`inEventLoop(Thread)`方法判断当前任务调用的线程是否为 EventLoop 本身的线程(即该任务是否直接分配给 EventLoop 本身的)，如果是则直接由 EventLoop 线程执行，否则 EventLoop 会将其加入自己独立的任务队列中

### ChannelHandler 和 ChannelPipeline

ChannelHandler 是消息处理器，负责处理 C/S 接收和发送的数据，它充当了所有处理入站和出站数据的应用程序逻辑的容器。

Channel 被创建时，自动分配到专用的 ChannelPipeline 中，ChannelPipeline 由多个 ChannelHandler 组成，每个 ChannelHandler 处理完后传给下一个，入站和出站`ChannelHandler`可以被安装到同一个`ChannelPipeline`中。

在Netty中，有两种发送消息的方式。你可以直接写到`Channel`中，也可以写到和`Channel-Handler`相关联的`ChannelHandlerContext`对象中。前一种方式将会导致消息从`Channel-Pipeline`的尾端开始流动，而后者将导致消息从`ChannelPipeline`中的下一个`Channel- Handler`开始流动。

#### 生命周期

- Channel：
  
  - 未注册到 EventLoop
  
  - 已注册
  
  - 未与远程节点连接
  
  - 已连接

- ChannelHandler：加入/移除 ChannelPipeline、异常
  
  - ChannelInboundHandler：
    
    - Channel 从 EventLoop 注册/注销
    
    - Channel 处于/离开活动状态
    
    - Channel 的上一个读操作已完成
    
    - 从 Channel 读取数据
    
    - Channel 的可写状态发生改变
    
    - 事件触发器
  
  - ChannelOutboundHandler：可以按需推迟操作或事件
    
    - Channel 绑定到本地地址
    
    - Channel 连接到/断开远程节点
    
    - 关闭 Channel
    
    - 从 Channel 读取数据
    
    - 通过 Channel 将入队数据冲刷到远程节点
    
    - 通过 Channel 将数据写入远程节点

#### ChannelHandler 适配器

  可以使用`ChannelInboundHandlerAdapter`和`ChannelOutboundHandlerAdapter`类作为自己的`ChannelHandler`的起始点。这两个适配器分别提供了`ChannelInboundHandler`和`ChannelOutboundHandler`的基本实现。通过扩展抽象类`ChannelHandlerAdapter`，它们获得了它们共同的超接口`ChannelHandler`的方法。

#### 资源管理

每当通过调用`ChannelInboundHandler.channelRead()`或者`ChannelOutbound- Handler.write()`方法来处理数据时，你都需要确保没有任何的资源泄漏。

为了帮助你诊断潜在的（资源泄漏）问题，Netty提供了`class ResourceLeakDetector`，定义了 4 中泄露检测级别：

- DISABLED：禁用泄露检测

- SIMPLE：使用 1% 的默认采样率检测并报告任何发现的泄露

- ADVANCED：使用默认采样率，报告发现的任何泄露和对应的消息被访问的位置

- PARANOID：对每次访问都进行采样

如果一个消息被消费或者丢弃了，并且没有传递给`ChannelPipeline`中的下一个`ChannelOutboundHandler`，那么用户就有责任调用`ReferenceCountUtil.release()`。如果消息到达了实际的传输层，那么当它被写入时或者`Channel`关闭时，都将被自动释放。

#### ChannelPipeline

每一个新创建的`Channel`都将会被分配一个新的`ChannelPipeline`。这项关联是永久性的；`Channel`既不能附加另外一个`ChannelPipeline`，也不能分离其当前的。

`ChannelHandler`可以通过添加、删除或者替换其他的`ChannelHandler`来实时地修改`ChannelPipeline`的布局。

#### ChannelHandlerContext

`ChannelHandlerContext`代表了`ChannelHandler`和`ChannelPipeline`之间的关联，每当有`ChannelHandler`添加到`ChannelPipeline`中时，都会创建`ChannelHandlerContext`。`ChannelHandlerContext`的主要功能是管理它所关联的`ChannelHandler`和在同一个`ChannelPipeline`中的其他`ChannelHandler`之间的交互。

如果调用`Channel`或者`ChannelPipeline`上的这些方法，它们将沿着整个`ChannelPipeline`进行传播。而调用位于`ChannelHandlerContext`上的相同方法，则将从当前所关联的`ChannelHandler`开始，并且只会传播给位于该`ChannelPipeline`中的下一个能够处理该事件的`ChannelHandler`。

`ChannelHandlerContext`和`ChannelHandler`之间的关联（绑定）是永远不会改变的，所以缓存对它的引用是安全的；

### ChannelFuture

Netty 的方法都是异步的(基于 NIO，但通过 Reactor 模型实现异步)，故需要 ChannelFuture 实现回调

通过 ChannelFuture 接口的 addListener() 方法注册一个 ChannelFutureListener，当操作执行完成时，监听就会自动触发返回结果

所有属于同一个`Channel`的操作都被保证其将以它们被调用的顺序被执行。

### Bootstrap

引导一个应用程序是指对它进行配置，并使它运行起来的过程

## 传输

因为Netty为每种传输的实现都暴露了相同的API，所以无论选用哪一种传输的实现，你的代码都仍然几乎不受影响。在所有的情况下，传输的实现都依赖于`interface Channel`、`ChannelPipeline`和`ChannelHandler`。

### 传输 Api

传输API的核心是`interface``Channel`，它被用于所有的I/O操作。每个`Channel`都将会被分配一个`ChannelPipeline`和`ChannelConfig`。`ChannelConfig`包含了该`Channel`的所有配置设置，并且支持热更新。由于特定的传输可能具有独特的设置，所以它可能会实现一个`ChannelConfig`的子类型。

Netty内置了一些可开箱即用的传输：

- NIO：提供了一个所有I/O操作的全异步的实现。它利用了自NIO子系统被引入JDK 1.4时便可用的基于选择器的API。

- OIO：Netty利用了`SO_TIMEOUT`这个`Socket`标志，它指定了等待一个I/O操作完成的最大毫秒数。如果操作在指定的时间间隔内没有完成，则将会抛出一个`SocketTimeout Exception`。Netty将捕获这个异常并继续处理循环。在`EventLoop`下一次运行时，它将再次尝试。

- Epoll：用于Linux的本地非阻塞传输

- Embedded：可以将一组`ChannelHandler`作为帮助器类嵌入到其他的`ChannelHandler`内部。通过这种方式，你将可以扩展一个`ChannelHandler`的功能，而又不需要修改其内部代码。

- Local：用于在同一个JVM中运行的客户端和服务器程序之间的异步通信。

## Reactor 线程模型

Reactor 模型基于事件驱动，适用于处理海量 I/O 事件，主要由 Reactor 与 Handler 组成：

- Reactor 线程：负责建立连接、监听 IO 事件、IO 事件读写、将事件分发到 Handlers 处理器
- Handlers 处理器：非阻塞地执行业务处理逻辑

### 事件驱动

以事件作为连接点，当有 I/O 事件准备就绪时，以事件的形式通知相关线程进行读写，进而业务线程可以直接处理数据，类似回调操作

在这种模式下，线程工作时必有数据可操作执行，不会在 I/O 等待上浪费资源

### 单线程 Reactor

所有 I/O 操作(建立连接、数据读写、事件分发)、业务处理都由一个线程完成：

- 逻辑简单、易于实现
- 单线程支持的连接数有限，对 CPU 的负载大
- 一个事件阻塞或处理时间较长时，后续的事件无法执行，会导致请求积压与超时
- 线程在进行 I/O 事件处理时会阻塞其他请求的接收
- I/O 线程长期满负荷对服务器性能影响大

Reactor 与 Handler 在同一个线程中，当 Handler 被阻塞时，其他所有 Handler 都无法执行，且监听新请求的 acceptor 也被阻塞，可能导致服务器无响应

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/28f4e7aaf2344cef95cddb6aaf353aca~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

### 多线程模型

将业务逻辑处理与请求处理分离，即将 Reactor 线程与 Handlers 处理器分开，由一个 Reactor 线程处理连接建立、I/O 读写操作、转发等，由线程池中的多个 Handlers 线程并发处理业务逻辑

因为 Reactor 线程仍为单线程模式，故在并发连接量较高的情况下存在性能问题(建立连接、监听 IO 就绪事件的开销)，且当某个连接通过系统调用读取数据时，Reactor 线程会被阻塞，无法完成其他事件

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/42464091af6e44d48ce240ca9616f56c~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

### 主从多线程

Reactor 线程由多个线程组成，每个 Reactor 线程都有独立的 Selector 对象，主 Reactor 仅负责处理客户端连接的 Accept 事件，连接建立后将新创建的连接对象注册至从 Reactor，再由从线程分配线程池中的 I/O 线程与连接绑定，负责连接生命周期内所有 I/O 事件

- mainReactor：只负责接收连接请求，将连接对象注册到 subReactor
- subReactor：负责更为耗时的 I/O 事件监听，利用多核 CPU 的特性，使更多就绪的 I/O 事件及时处理

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f7808a2690294fdcb17293c8d6c69b81~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

### Netty 线程模型

Netty 线程模型基于 Reactor 模型，通过 NioEventLoopGroup 线程池实现：

- bossGroup：类似 Reactor，负责接收请求
- workerGroup：类似 Handler，负责具体的处理

### Netty 启动过程

#### 服务端

1. 创建 bossGroup 与 workerGroup 对象实例
2. 创建启动引导类 ServerBootstrap
3. 通过 group() 方法为引导类配置两大线程组，确定线程模型(主从多线程)
4. 通过 channel() 方法给引导类 ServerBootstrap 指定 IO 模型为 NIO
5. 通过 childHandler() 方法给引导类创建一个 ChannelInitializer，指定服务端消息的业务处理逻辑 HelloServerHandler 对象
6. 调用 ServerBootstrap 类的 bind() 方法绑定端口

#### 客户端

1. 创建 NioEventLoopGroup 对象实例
2. 创建客户端启动引导类 Bootstrap
3. 通过 group() 方法给引导类 Bootstrap 配置线程组
4. 通过 channel() 方法给引导类指定 IO 模型为 NIO
5. 通过 handler() 方法给引导类创建一个 ChannelInitializer，指定客户端消息的业务处理逻辑 HelloClientHandler 对象
6. 调用 Bootstrap 类的 connect() 方法进行连接

## TCP 粘包/拆包

TCP 面向流，没有边界，而操作系统在发送 TCP 数据时会通过缓存区来进行优化

- 粘包：一次发送的数据量小于缓冲区，TCP 会将多个请求合并为同一个请求发送
- 拆包：一次发送的数据量大于缓冲区，TCP 会将一个请求拆分为多个请求发送

![](https://pic1.zhimg.com/80/v2-9072eb69d4ec6d74d95f4f633df87f10_720w.webp)

### 常用解决方案

- 填充：需要发生粘包时，改为使用特殊字符(0 或 null)进行填充，保证每个包的大小一致
- 标识符：需要拆包时，标记拆分的每个包，收到包后通过标记重新合并为完整的包
- 头信息：将消息分为头部和消息体，头部保存整个消息的长度，只有读取到足够长度的消息后才算读取完整的消息
- 自定义协议

### Netty 解决方案

Netty 提供了 Decoder 解决粘包与拆包的问题

- LineBasedFrameDecoder：每个数据包之间以换行符作为分隔，接收时一次遍历 ByteBuf 中的可读字节，若有换行符则进行截取
- DelimiterBasedFrameDecoder：自定义分隔符
- FixedLengthFrameDecoder：固定长度解码器，按照指定的长度进行拆包，不足时由空格补全
- LengthFieldBasedFrameDecoder：长度域解码器，根据发送的数据中消息长度相关参数进行拆包
- 自定义序列化编解码器

## 长连接、心跳机制

TCP 的三次握手与四次挥手耗费大量资源，在使用短连接时，每次请求都需要进行连接，且完成后都会断开连接，故需要使用长连接，C/S 连接不会主动关闭，在频繁请求资源的场景可以降低开销

由于长连接不会主动关闭，故需要引入心跳机制保证连接双方都存活，避免无效连接长期存在：

- PING-PONG：连接双方发送与响应特殊的数据包，由此确认对方是否存活

TCP 自带的长连接与心跳机制不够灵活，Netty 通过 IdleStateHandler 实现心跳机制

## 零拷贝

通过映射方法令程序直接操作内存地址，而不需要先拷贝到特定区域修改后再回写，主要用于避免用户态与内核态的切换开销

Netty 中使用零拷贝技术优化数据操作：

- 堆外内存：避免 JVM 堆内存与堆外内存的拷贝
- 使用 CompositeByteBuf 类将多个 ByteBuf 合并为一个逻辑上的 ByteBuf，避免多个 ByteBuf 之间的拷贝
- 通过 Unpooled.wrappedBuffer 将 byte 数组包装成 ByteBuf 对象
- ByteBuf 的 slice 操作将 ByteBuf 分解为多个共享同一存储区域的 ByteBuf，避免内存拷贝
- 通过 FileRegion 包装的 FileChannel#transferTo() 实现文件传输，直接将文件缓冲区的数据发送到目标 Channel，避免通过循环 write 方式导致的内存拷贝问题
