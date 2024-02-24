# Java 并发

## 进程

进程是程序的一次执行过程(系统运行一次程序即是进程的创建、运行、消亡)，是**系统运行程序的基本单位**，因此进程是动态的

## 线程

线程是比进程更小的执行单位，一个进程在执行时可能生成多个线程，同类线程之间**共享进程的堆和方法区**的同时也拥有自己的**程序计数器、本地方法栈、虚拟机栈**，所以线程的产生与切换的开销比进程小得多

Java 程序天生就是多线程程序

## Java 线程 vs 操作系统线程

Java 2 前，Java 的线程由绿色线程(Green Thread) 实现，即 JVM **基于操作系统的用户线程**模拟多线程的运行，而不依赖与操作系统。但用户线程的使用有限制(无法直接使用操作系统提供的功能如异步 I/O，只能在一个内核线程上运行无法利用多核)

Java 2 后，Java 的线程改为原生线程(Native Thread)实现，即 JVM **直接使用操作系统原生的内核线程**，由操作系统内核进行线程的调度与管理

- 用户线程：由用户空间程序管理和调度的线程，运行在用户空间(应用程序)，创建和切换的成本较低，但有限制
- 内核线程：由操作系统内核管理和调度的线程，运行在内核空间，能够利用多核，但创建与切换的成本高

Java 线程本质上是操作系统线程

## 线程 vs 进程 (JVM)

一个进程可以有多个线程，进程之间相互独立，同类线程共享进程的堆与方法区(Java 8 后为元空间)资源，每个线程又有自己私有的程序计数器、虚拟机栈、本地方法栈

进程的创建与切换的开销较大，线程的创建与切换的开销较小，但线程共享资源不利于资源的管理和保护

### 程序计数器

- 代码的流程控制：通过改变程序计数器来依次执行指令(Java 方法，执行 native 方法时为 undefined)
- 多线程下的还原：用于标识线程在字节码中走到哪一步，便于线程切换时还原

### 虚拟机栈与本地方法栈

线程在方法调用时会为当前状态(局部变量表，操作数栈，常量引用池等)创建栈帧并入栈，在调用完成后再出栈还原状态

虚拟机栈用于 Java 方法，而本地方法栈用于 native 方法，在 Hotspot JVM 中将两者合二为一

虚拟机栈与本地方法栈的线程私有**保证线程中的局部变量不会被其他线程访问**

### 堆与方法区

堆与方法区是所有线程共享的资源

- 几乎所有创建的对象都存储在堆中(JIT 标量替换后会创建在栈中)
- 方法区用于存储已加载的类信息、常量、静态变量、JIT 缓存等

## 创建线程

继承 Thread 类，实现 Runnable 接口，实现 Callable 接口都能创建线程，但严格来说都是需要通过 new Thrad().start() 来创建

Runnable，Callable 对象只是提供给线程执行的任务

## 线程的生命周期与状态

- NEW：线程刚被创建
- RUNNABLE：线程准备好执行所需的资源的运行状态
  - JVM 将线程的 READY 与 RUNNING 状态合并，因为如今 CPU 的时间分片很小，即每个线程只会在 CPU 执行很短的时间(10-20ms)，故线程在 READY 与 RUNNING 状态之间切换十分快，区分两种状态的意义不大
- BLOCKING：线程被阻塞，等待锁(会主动获取锁)，由于获取不到锁而被动进入阻塞状态
- WAITING：线程休眠，需要等待通知(唤醒)才会尝试获取锁，由 wait() 方法主动进入等待状态
- TIME_WAITING：线程定时，在指定时间后自动返回而不会一直等待(WAITING)
- TERMINATED：线程终止

![](https://mmbiz.qpic.cn/mmbiz_png/GLeh42uInXQFkYWhfJUhibHcHb4VoVp9Je8FwJvhEDcn1zwhktV7iaTjkozU2LoiaqO3XP5fDibWichGyJ5uwN3cqnQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

## 线程上下文切换

线程让出 CPU 资源：

- 线程进入 WAITING/TIME_WAITING 状态(wait()，sleep())
- CPU 调度线程
- 线程阻塞(请求 I/O)
- 线程完成或被终止

在前三种情况，线程保存在执行过程中的信息(程序计数器、栈)，即上下文后主动让出 CPU 资源，等待线程下次占用时还原现场，再加载下一个线程的上下文

由于保存/恢复信息有额外的开销，故频繁切换线程会降低效率

## 线程死锁

线程死锁即两个线程都因为等待对方持有的资源释放而无限期地处于阻塞状态

### 死锁产生的必要条件

- 互斥：资源只能被一个线程持有
- 请求与保持：线程不会主动放弃持有的部分资源
  - 破坏：线程只会一次性请求所有资源
- 不剥夺：线程持有的资源不会被其他线程剥夺，只能等待线程主动释放
  - 破坏：线程在阻塞时主动放弃持有的资源
- 循环等待：线程之间的等待关系构成环
  - 破坏：人为对资源的请求排序

### 避免死锁

通过算法为资源的分配进行评估，使其进入安全状态(存在一种分配顺序，使每个线程最终都能顺利完成)

## sleep() vs wait()

两者都会让线程进入 WAITING/TIME_WAITING 状态

- 锁：wait 会主动释放锁；sleep 不会释放锁
- 唤醒：wait 需要等待同一对象的 notify/notifyAll 唤醒，或使用 wait(time) 定时唤醒；sleep 会定时唤醒
- 用法：wait 用于线程间通信；sleep 用于暂停线程
- 从属：wait 从属于 Object 类的本地(native)方法；sleep 从属于 Thread 类的静态本地方法
  - wait() 让获得对象锁的线程实现等待，会自动释放当前线程占有的对象锁
    - 对象锁：每个对象 (Object) 都拥有对象锁(JVM 负责跟踪对象被加锁的次数)，若要释放当前线程占有的对象锁并进入 WAITING 状态，就需要操作对应对象(Object)而非当前的线程(Thread)
  - sleep() 只让当前线程暂停而不需要释放锁，故其操作对象只涉及线程

## 直接调用 Thread 类的 run 方法

调用 Thread 类的 run 方法会执行线程体，但不会新建线程(内部调用 Runnable 的 run 方法)，即线程体会运行在 main 线程中，即使由 new Thread 对象调用的 run 方法

需要通过 Thread 对象的 start 方法才能真正启动一个线程并使线程进入就绪状态后获取时间片运行

调用 start() 方法可以启动线程并使线程进入就绪状态，直接运行 run() 方法不会以多线程方式执行(main 线程中执行)

## Volatile

### 如何保证可见性

- 读操作：volatile 修饰的变量在读操作前会将工作内存的变量设为无效，再从共享内存中读取
- 写操作：volatile 修饰的变量的写操作会直接将结果写入主存，不保证线程安全性(n 个线程执行 cnt++ 操作的结果不一定是 cnt + n，因为线程只会在一开始的读操作保证当前的数据是最新的)

### 禁止指令重排序

volatile 修饰的变量进行读写操作时会通过插入内存屏障来禁止重排序

#### 单例模式

````java
public class Singleton {
    
    private volatile static Singleton instance;
    
    public static Singleton getInstance(){
        // 判断单例对象是否已创建
        if(instance == null){
            // 类对象加锁
            synchronized (Singleton.class) {
                // 加锁前已被其他线程创建
                if(instance == null){
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
    
}
````

volatile 修饰 instance：

- 由于 new Singleton() 分为三步执行：
  1. 在堆中分配内存空间
  2. 初始化对象
  3. 将分配的内存空间地址赋值给栈中变量
- 其中 2，3 步可能发生重排序，即先将内存空间地址赋值给栈
- 若此时为多线程环境，则可能线程 A 执行 1，3 后，instance 为非空但未初始化，线程 B 此时判断 instance 非空并直接返回未初始化的 instance

### 保证原子性

volatile 可以保证单个变量的可见性，不能保证对变量操作的原子性

````java
public volatile static int inc = 0;

public void increase() {
    // 非原子性操作
    // 1. 读取 inc
    // 2. 修改 inc
    // 3. 写回 inc
    inc++;
}

public void exc() {
    ExecutorService threadPool = Executors.newFixedThreadPool(5);
    for(int i = 0; i < 5; i++) {
        threadPool.execute(() -> {
            for(int j = 0; j < 500; j++){
                increase();
            }
        })
    }
}
````

此时 inc 的值不一定为 2500，因为 volatile 只会保证线程在进行读操作时获取到的是最新的数据，并不能保证读之后其他线程的修改是可见的，即：

1. 线程 A 读取 inc，此时的 inc 是最新的
2. 线程 B 写回 inc，此时的 inc 为 inc + 1，但线程 A 仍使用原本的 inc
3. 线程 A 将 inc + 1 写回主存，但正确结果应为 inc + 2

可以使用 synchronized、ReentrantLock、AtomicInteger 保证原子性

## 悲观锁

悲观锁即每次读都假设数据会被修改，故**每次操作都会上锁**，从而阻塞其他线程的操作，即严格要求共享资源只被一个线程使用(读/写)

Java 中的 synchronized、ReentrantLock 等独占锁就是悲观锁的实现

```java
public void performSynchronizedTask() {
    synchronized(this) {
        // 需要同步的操作
    }
}

private Lock lock = new ReentrantLock();
lock.lock();
try {
    // 需要同步的操作
} finally {
    lock.unlock();
}
```

高并发场景下，激烈的锁竞争会造成大量阻塞，从而引起大量的线程上下文切换，增加性能开销

悲观锁也存在死锁问题

故悲观锁适用于写多读少的场景，因为悲观锁的开销是固定的(所有操作都加锁)，可以避免频繁失败与重试带来的影响

## 乐观锁

乐观锁假设读操作时一定不会发生修改，即**不加读锁，只在写操作执行时验证数据是否被修改**(版本号，CAS 算法等)

Java 中的原子变量类(AtomicInteger、LongAddr)就是使用了乐观锁(CAS)的一种实现

在高并发场景，不加锁、不阻塞的乐观锁的性能远高于悲观锁，但当冲突频繁发生时(写操作居多)会频繁失败与重试，也会对性能产生影响

故乐观锁适用于读多写少的场景，因为不需要为读操作加锁与阻塞，只有在写操作时可能失败与重试

### 乐观锁实现

#### 版本号

在数据库表中加入 version 字段，表示数据修改的次数(每次修改加一)，当更新数据时只有与当前数据库中的版本一致才允许更新，否则重试更新操作

#### CAS 算法

Compare And Swap，即使用预期值(E)与当前值(V)比较，两值相等时才会更新需要写入的新值(N)

CAS 是依赖于 CPU 原子指令的原子操作，由 C++ 内联汇编的形式实现，通过 JNI 调用，故可以保证比较的原子性，且 CAS 算法与操作系统及 CPU 相关

### 问题

#### ABA

若用于比较的当前值(V)在比较期间发生 A->B->A 的变化，则在乐观锁看来此变量未被改变过

解决思路：为 V 添加版本号/时间戳，确保 V 的唯一性

#### 循环时间长开销大

CAS 使用自旋操作进行重试，即一直重试到成功，故长时间不成功会带来巨大开销

解决思路：使用 CPU 提供的 pause 指令延迟重试，并且避免在退出循环时因内存顺序冲而引起 CPU 流水线被清空

#### 只能保证一个共享变量的原子操作

CAS 只对单个共享变量有效，操作涉及多个共享变量时无效

从 Java 5 开始使用 AtomicReference 类保证引用对象间的原子性，可以把多个变量放入一个对象中进行 CAS 操作，即可以利用锁/AtomicReference 类把多个共享变量合并成一个共享变量操作

## Synchronized

synchronized 关键字用于保证多线程访问共享变量时的同步性，可以保证同一时刻只有一个线程执行代码块/方法

- Java 6 前：synchronized 是重量级锁，底层基于操作系统的 Mutext Lock 互斥锁实现，Java 的线程挂载于操作系统的线程，每次线程的阻塞/唤醒操作都需要操作系统 用户态->内核态->用户态 的转换
- Java 6 后，synchronized 进行了大量优化，引入了自旋锁，轻量级锁，锁销除，锁粗化，偏向锁等技术来减少开销

偏向锁的引入增加了 JVM 的复杂度，且性能提升不大，故在 Java 15 中默认关闭，在 Java 18 中废弃

### 使用

- 实例方法：给对象实例加锁
- 静态方法：给当前类加锁
  - 实例方法与静态方法不互斥：线程 A 调用实例对象的非静态 synchronized 方法，获取实例对象的锁；线程 B 调用实例对象的静态 synchronized 方法，获取当前类的锁
- 代码块
  - synchronized(object)：进入代码块前获取给定对象的锁
  - synchronized(class)：进入代码块前获取给定类的锁

synchronized 不能修饰构造函数，因为构造函数本身是线程安全的，即每个线程 new 的对象是独立的，并不会产生冲突，但构造函数的参数可能是共享的，故构造函数内的代码块可以被 synchronized 修饰

### 底层实现

synchronized 关键自底层原理属于 JVM 层面，其本质为对象监视器 monitor 的获取

#### 代码块

synchronized 同步语句块的实现使用的是 monitorenter 和 monitorexit 指令，其中 monitorenter 指令指向同步代码块的开始位置，monitorexit 指令则指明同步代码块的结束位置

HotSpot 中，Monitor 是基于 C++ 的 ObjectMonitor 实现的，每个对象都内置了一个 ObjectMonitor 对象。wait/notify 等方法也依赖于 monitor 对象，所以只有在同步的块或方法中才能调用 wait/notify 等方法

执行 monitorenter 时会尝试获取对象的锁，若锁的计数器为 0 则表示锁可以被获取，获取后将锁计数器加一

![](https://oss.javaguide.cn/github/javaguide/java/concurrent/synchronized-get-lock-code-block.png)

只有拥有对象锁的线程能够执行 monitorexit 指令来释放锁，在执行 monitorexit 指令后，将锁计数器设为 0，表明锁被释放，其他线程可以尝试获取锁

![](https://oss.javaguide.cn/github/javaguide/java/concurrent/synchronized-release-lock-block.png)

如果获取对象锁失败，那当前线程就要阻塞等待，直到锁被另外一个线程释放为止

#### 方法

synchronized 修饰方法并不会产生 monitorenter 与 monitorexit 指令，而是由 ACC_SYNCHORIZED 访问标识指明该方法为一个同步方法，JVM 由此辨别同步方法，从而执行相应的同步调用

对于实例方法，JVM 会尝试获取实例对象的锁，对于静态方法，JVM 会尝试获取当前类对象的锁

### 优化

在 Java 6 后，synchronized 底层引入了自旋锁、适应性自旋锁、偏向锁、锁消除、锁加粗、轻量级锁等技术来减少锁操作的开销

锁主要存在四种状态：

- 无锁状态
- 偏向锁状态
- 轻量级锁状态
- 重量级锁状态

它们随着竞争的激烈程度而逐渐升级，且该升级不可逆，由此来提高获取锁和释放锁的效率

### synchronized vs volatile

两个关键字都能确保变量的可见性与有序性

- 修饰范围：volatile 只能修饰变量；synchronized 能修饰方法与代码块
- 原子性：volatile 只能保证对单个变量的可见性；synchronized 能保证同一时间只有一个线程执行方法/代码块
- 开销：volatile 更加轻量级，主要用于解决共享变量在多个线程之间的可见性；synchronized 解决多个线程之间访问资源的同步性

## ReentrantLock

ReentrantLock 实现了 Lock 接口，是一个可重入且独占式的锁，和 synchronized 关键字类似，但 ReentrantLock 更灵活、更强大，增加了轮询、超时、中断、公平锁、非公平锁等高级功能

ReentrantLock 中有一个内部类 Sync，Sync 继承自 AQS，添加/释放锁的大部分操作实际上都是在 Sync 中实现的。Sync 有公平锁 FairSync 和非公平锁 NonfairSync 两个子类

ReentrantLock 默认使用公平锁，也可以通过构造器来显式地指定使用公平锁

### 公平锁 vs 非公平锁

- 公平锁：锁被释放后，先申请的线程先得到锁(先到先得)，为了保证时间上的公平性，会频繁切换上下文，导致性能较差
- 非公平锁：锁被释放后，线程按其他优先级顺序/随机获取到锁，性能更高，但会有饿死的问题

### synchronized vs ReentrantLock

#### 可重入锁

两者都是可重入锁，即线程可以再次获取到自己的内部锁

```java
public class SynchronizedDemo {
    public synchronized void m1() {
        m2();
    }
    
    public synchronized void m2() {
        
    }
}
```

同一线程在调用 m1 时获取到当前对象锁，执行 m2 时可以再次获取到当前对象的锁

若 synchronized 不可重入，则因为该对象的锁被当前线程持有且无法释放，导致线程在执行 m2 时获取锁失败，出现死锁问题

#### 底层实现

- synchronized 关键字依赖于 JVM 中的 monitor 对象，实质是利用 C++ 的 ObjectMonitor 对象，其优化也由 JVM 实现
- ReentrantLock 依赖于 JDK 层面的实现

#### 功能

ReentrantLock 新增了一些高级功能：

- 等待可中断：ReentrantLock 通过 lock.lockInterruptibly() **令正在等待的线程主动放弃等待**，改为处理其他事情
  - 可中断锁：获取锁的过程中可以主动放弃等待
  - 不可中断锁：一旦开始申请锁，线程就会一直等待至获取到锁，synchronized 为不可中断锁
- 可实现公平锁：ReentrantLock 默认为非公平锁，可以通过构造函数指定为公平锁
- 可实现选择性通知(锁可以绑定多个条件)：synchronized 关键字与 wait()、notify()、notifyAll() 方法相结合实现等待/通知机制；ReentrantLock 通过 Condition 接口与 newCondition() 方法实现
  - Condition：可以理解为一个 Monitor 对象，一个 Lock 能够挂载多个 Condition，不同的线程能够注册在不同的 Condition 中，并由此选择性地通知(notify)指定的线程
    - 在 synchronized 中，notify() 唤醒的线程由 JVM 决定，所有线程都注册在同一个 Monitor 对象中，故 notifyAll() 的性能开销大

## ReentrantReadWriteLock

实现读写锁，读锁可以被多个线程持有(共享锁)，写锁只能被一个线程持有(独占锁)，从而实现读读不互斥，保证多个线程同时读的效率与写入操作时的线程安全，适用于读多写少的场景

- 存在线程持有读锁时(有线程在读)，无法获取写锁，但仍能获取读锁
- 存在线程持有写锁时(有线程在写)，无法获取读锁与写锁

## ThreadLocal

ThreadLocal 类用于为每个访问的线程提供一份副本，即每个访问 ThreadLocal 类的线程都会在工作内存中创建一份副本来获取/修改

### 原理

```java
public class Thread implements Runnable {
    // 与此线程有关的 ThreadLocal 值，由 ThreadLocal 类维护
    ThreadLocal.ThreadLocalMap threadLocals = null;
    
    // 与此线程有关的 InheritableThreadLocal 值，由 InheritableThreadLocal 类维护
    ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;
}
```

Thread 类中有一个 threadLocals 和一个 inheritableThreadLocals 变量，都是 ThreadLocalMap 类型(为 ThreadLocal 类定制的 HashMap)，只有当前线程调用 ThreadLocal 类的 set 和 get 方法时才能创建

```java
public void set(T value) {
    // 获取当前线程
    Thread t = Thread.currentThread();
    // 取出当前线程的内部变量表 ThreadLocalMap
    ThreadLocalMap map = getMap(t);
    if(map != null)
        map.set(this, value);
    else
        createMap(t, value);
}

ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}
```

**最终的变量是放在了当前线程的 ThreadLocalMap 中**，ThreadLocal 类可以通过 Thread.currentThread() 获取到当前线程对象后，直接通过 getMap(Thread t) 可以访问到该线程的 ThreadLocalMap 对象

每个 Thread 中都具备一个 ThreadLocalMap，而 ThreadLocalMap 可以存储以 ThreadLocal 为 key，Object 对象为 value 的键值对

![](https://pic2.zhimg.com/80/v2-adecb8b867ce06a962df2a3668563101_720w.webp)

### ThreadLocal 内存泄漏

#### 弱引用

只具有弱引用的对象拥有比软引用更短的生命周期，在垃圾回收线程扫描它管辖的内存区域时，一旦发现了只具有弱引用的对象，不管当前内存空间是否足够，都会回收

将弱引用和一个引用队列联合使用，当弱引用所引用的对象被垃圾回收，JVM 就会把这个弱引用加入到与之关联的引用队列中

#### 强引用

只要对象具有强引用，该对象就不能被回收

#### 内存泄漏原因

ThreadLocalMap 中的 key 是 ThreadLocal 的弱引用，value 是强引用，故当 ThreadLocalMap 没有被外部强引用的情况下，在垃圾回收时，key 会被清理掉，value 不会。此时会产生 key 为 null 的 Entry，GC 回收无法辨别，相应的内存就被泄漏

key 为弱引用：ThreadLocalMap 与线程绑定，若 key 为强引用，则当对应的 ThreadLocal 不再被使用时也不会回收，其对应的 value 对象也会随线程一直存在，直到线程销毁。将 key 设置为弱引用就可以保证不再用到的 ThreadLocal 会被及时回收

value 为强引用：若 value 为弱引用，则当其 Object 对象不存在外部强引用时就会被 GC 回收，此时 key 的 ThreadLocal 还存在，就会产生获取结果为 null 的情况

#### 解决

ThreadLocalMap 实现中，在调用 set()/get()/remove() 方法时，会清理掉 key 为 null 的 Entry，故在使用完 ThreadLocal 方法后应手动调用 remove() 方法

## 线程池

线程池就是管理一组线程的资源池，当有任务需要处理时可以直接分配给线程池中的线程，而不用在生成新的线程，当线程处理完任务后也可以直接等待下一个任务，而不需要进行销毁，由此可以减少线程创建/销毁的开销，提高资源的利用率

- 减少开销：不需要频繁进行线程的创建/销毁
- 加快响应：任务可以直接分配给等待的线程执行，而不需要进行创建
- 便于管理：所有线程都由线程池统一管理

### 创建线程池

- ThreadPoolExecutor 构造函数
- Executor 框架工具类 Executors (JDK 内置的线程池)：
  - FixedThreadPool/SingleThreadExecutor：使用无界的 LinkedBlockingQueue，最大长度为 Integer.MAX_VALUE，可能 OOM
  - CachedThreadPool：使用同步队列 SynchronousQueue，最大长度为 Integer.MAX_VALUE
  - ScheduledThreadPool/SingleThreadScheduledExecutor：使用无界的延迟阻塞队列 DelayedWorkQueue，任务队列最大长度为 Integer.MAX_VALUE
  - 在任务数量多且处理速度满的情况下，可能会堆积大量请求，从而导致 OOM

### 线程池参数

ThreadPoolExecutor:

- corePoolSize：核心线程数，即任务队列未满时最大可以同时运行的线程数
  - keepAliveTime：线程等待的最大时长，直到线程池中的数量 == 核心线程数时才不再回收
- maximumPoolSize：任务队列已满时，同时可运行的线程数
  - maximumPoolSize = corePoolSize + 非核心线程数
- workQueue：正在工作的线程数 == 核心线程数时，将新任务暂存于工作队列

### 线程池饱和策略

任务队列已满时对新任务的处理：

- AbortPolicy：拒绝处理并抛出异常
- CallerRunsPolicy：直接在调用者的线程中运行被拒绝的任务，若调用者已关闭，则丢弃任务。该策略会降低新任务的提交速度(调用者需要自己先处理任务)，影响程序整体性能，适用于能够接受延迟且要求每个任务都被执行的场景
- DiscardPolicy：直接丢弃
- DiscardOldestPolicy：直接丢弃任务队列中等待最久的任务

### 阻塞队列

- LinkedBlockingQueue：最大值为 Integer.MAX_VALUE 的无界队列
  - FixedThrealPool：核心线程数 == 最大线程数
  - SingleThreadExector：只能创建一个线程
- SynchronousQueue：同步队列，没有容量，不存储元素，将提交的任务分配给空闲线程或创建新的线程处理，即线程是无限扩展的
  - CachedThreadPool
- DelayedWorkQueue：延迟队列，内部元素按延迟的时间长短排序，只有到达时间的才能出列(堆)，队列满时会自动扩容(增加 1/2，最大 Integer.MAX_VALUE)，即永远不会阻塞
  - ScheduledThreadPool/SingleThreadScheduledExecutor

### 处理任务流程

1. 当前线程数 < 核心线程数：创建新线程执行任务
2. 当前线程数 >= 核心线程数 && 当前线程数 < 最大线程数：将任务放入工作队列中等待
3. 任务队列已满 && 当前线程数 < 最大线程数：新建线程执行任务
4. 当前线程数 == 最大线程数：调用饱和策略处理

### 线程池大小

线程池过小会导致大量的请求堆积/丢弃

线程池过大会造成频繁的线程上下文切换(大量的线程同时抢夺 CPU 资源)

- CPU 密集型(n + 1)：任务主要消耗 CPU 资源，可以将线程池大小设置为 CPU 核心数 + 1，多出的线程用于防止线程偶发的缺页中断或其他原因导致的暂停
- I/O 密集型(2n)：大量的 I/O 交互会导致许多线程阻塞，等待 I/O 期间不需要占用 CPU 资源，可以主动放弃给其他线程使用

### 任务优先级执行的线程池

使用 PriorityBlockingQueue 构造 workQueue

PriorityBlockingQueue 支持优先级的无界队列，但不支持阻塞操作

#### 实现

- 提交到队列的线程需要可比较：
  - 实现 Comparable 接口，重写 compareTo 方法
  - 构造函数中传入 Comparator 对象指定排序规则

#### 问题

- PriorityBlockingQueue 无界，可能堆积大量请求
  - 通过重写 offer 方法，手动设置容量
- 优先级导致饿死
  - 优化设计，令长期等待的低优先级任务重新入队并提高优先级
- 排序性能

## Future

Future 类的核心思想是异步调用，即将耗时长的任务交由子线程异步执行，在需要用到任务结果时再通过 Future 类获取其返回值，如此就不需要一直原地等待耗时任务执行完成，提高了程序的效率

Java 中的 Future 类是一个泛型接口，主要定义了以下功能：

- 取消任务
- 判断任务是否被取消
- 判断任务是否已经执行完成
- 获取任务执行结果

### FutrueTask

FutureTask 提供了 Future 接口的基本实现，常用于封装 Callable 和 Runnable，具有取消任务、查看任务是否执行完成以及获取任务执行结果的方法

FutureTask 也实现了 Runnable 接口，因此可以作为任务直接被线程执行

FutureTask 相当于对 Callable 进行封装，管理任务执行情况，存储 Callable 的 call 方法的任务执行结果

### CompletableFuture

Future 类不支持异步任务的编排组合(将多个异步任务串联起来，组成一个完整的链式调用)，获取计算结果的 get() 方法为阻塞调用

CompletableFuture 类用于提供更加强大的 Future 接口实现，其实现了 CompletionStage 接口与 Runnable 接口

CompletionStage 接口描述了一个异步计算的阶段，很多计算可以分成多个阶段或步骤，此时可以通过它将所有步骤组合起来，形成异步计算的流水线

## AQS

AbstractQueueSynchronizer 即抽象队列同步器，该接口为锁与同步器的实现提供了许多通用的方法

### 原理

核心思想：线程在获取共享资源时，若资源空闲则直接获取并加锁；若已被占有，则需要引入一套阻塞等待与唤醒分配机制来等待资源被释放

CLH 队列是一个虚拟的双向队列(并不存在实例，只是逻辑上的关系)，将需要获取共享资源的线程封装成一个节点存储

- 等待状态
- 线程引用
- 前驱节点
- 后继结点

![](https://oss.javaguide.cn/p3-juejin/40cb932a64694262993907ebda6a0bfe~tplv-k3u1fbpfcp-zoom-1.png)

AQS 通过维护一个 int 变量 state 来表示资源的状态，线程获取到资源时会将 state + 1 表示加锁(可重入锁会不断叠加)，在解锁时将 state -1，直到 state 回到 0 时才能被其他线程获取，state 由 volatile 修饰以确保可见性

AQS 通过内置的线程等待队列完成线程的排队工作

![](https://oss.javaguide.cn/github/javaguide/java/CLH.png)

### Semaphore

信号量，用于规定有多少线程能获取共享变量的场景，若线程为 1 则退化为互斥锁

Semaphore 能够实现公平与非公平模式

Semaphore 使用简单，可以用于资源有明确访问数量限制的场景(限流)，但一般用于单机模式，实际项目中使用 Redis + Lua

#### 原理

Semaphore 是一种共享锁的实现，通过 AQS 的 state 来标识有多少个线程能获取资源。线程在获取资源(semaphore.acquire())时会通过 CAS 尝试修改 state -1，成功则获取到资源；解锁(semaphore.release())时也通过 CAS 操作令 state + 1，再唤醒等待的线程通过 CAS 获取锁

### CountDownLatch

类似于屏障，允许 count 个线程阻塞在一个地方，直至所有线程都执行完毕

CountDownLatch 是一次性的，即 count 只在初始化时赋值，之后不可修改，且 CountDownLatch 执行完后不可再次使用

#### 原理

CountDownLatch 用 count 作为 AQS 的 state 值，每个线程调用 countDown() 方法时通过 CAS 操作将 state - 1，调用 await() 方法时若 state 值不为零(仍有线程未执行完毕)则阻塞线程，直到 count 个线程调用了 countDown() 方法或 await() 方法被中断才会唤醒

#### 使用场景

多线程处理多个文件，需要将所有文件处理完毕后才返回：

- 定义线程池与 count 为 6 的 CountDownLatch 对象
- 使用线程池处理任务，每个线程处理完后调用 countDown() 方法
- 调用 CountDownLatch 对象的 await() 方法，直到所有文件都执行完后才返回

````java
public class CountDownLatchExample1 {
    // 处理文件的数量
    private static final int threadCount = 6;

    public static void main(String[] args) throws InterruptedException {
        // 创建一个具有固定线程数量的线程池对象（推荐使用构造方法创建）
        ExecutorService threadPool = Executors.newFixedThreadPool(10);
        final CountDownLatch countDownLatch = new CountDownLatch(threadCount);
        for (int i = 0; i < threadCount; i++) {
            final int threadnum = i;
            threadPool.execute(() -> {
                try {
                    //处理文件的业务操作
                    //......
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    //表示一个文件已经被完成
                    countDownLatch.countDown();
                }

            });
        }
        countDownLatch.await();
        threadPool.shutdown();
        System.out.println("finish");
    }
}

````

改进：

- 通过 CompletableFuture 类提供的多线程友好方法
- 任务过多时循环添加任务

### CyclicBarrier

功能更强大的 CountDownLatch，基于 ReentrantLock 类和 Condition 接口实现，能够让一组线程到达一个同步点时被阻塞，直到所有线程到达时才会唤醒

#### 原理

CyclicBarrier 内部通过一个 count 变量作为计数器，count 的初始值为 parties 属性的初始化值，每当一个线程到达栅栏调用 await() 方法通知 CyclicBarrier，count - 1，当 count == 0 表示所有线程都到达，则尝试执行构造方法中输入的任务

```java
public CyclicBarrier(int parties) {
    this(parties, null);
}

public CyclicBarrier(int parties, Runnable barrierAction) {
    if (parties <= 0) throw new IllegalArgumentException();
    // 拦截的线程数量
    this.parties = parties;
    this.count = parties;
    this.barrierCommand = barrierAction;
}

```

