## MySQL 的存储引擎

### InnoDB

InnoDB 是通用的存储引擎，在 MySQL 5.6 后作为默认的存储引擎

- 支持 ACID 事务，默认为可重读的事务级别
- 外键
- 行级锁
- 通过 MVCC 实现高并发
- 在磁盘中聚集存储，根据主键进行顺序存储

### MyISAM

MyISAM 在 MySQL 5.5.8 前作为默认的存储引擎

- 不支持事务
- 表级锁
- 支持全文索引
- 缓存索引而非数据

## 连接 MySQL

MySQL 的数据库连接是一个通信进程和数据库实例之间的通信，本质上是进程间通信：

- TCP/IP
- 命名管道和共享内存/Unix 域套接字：通信进程需要在一台服务器上

## InnoDB 体系架构

InnoDB 引擎有多个内存块，这些内存块组成了一个大的内存池，用于作为引擎和磁盘之的缓存(缓存频繁读取的数据与需要写入磁盘的脏数据)

### 后台线程

InnoDB 是多线程架构：

- Master Thread：核心线程，负责将缓冲池的数据写回磁盘，保证数据一致性
  - 脏页刷新
  - Undo Page 回收
  -  合并插入缓存
- IO Thread：InnoDB 使用大量的 AIO 处理写 IO 请求，IO Thread 用于负责 AIO 的回调处理
- Purge Thread：在事务提交后，undolog 不再被需要，Purge Thread 就是用于回收已使用并分配的 Undo Page
- Page Cleaner Thread：负责脏页刷新操作，减轻主线程负担

### 内存

#### 缓冲池

InnoDB 是基于磁盘的数据库系统，将数据以页为单位在磁盘存储与管理，为了平衡 CPU 与磁盘的性能差异，InnoDB 在内存中维护一个内存池作为缓冲

当需要读取数据时，CPU 会先从缓冲池中读取，如果没有命中则从磁盘读取并将数据 FIX 在缓冲池

修改数据时也是先对缓冲池中的数据进行修改，在到达 checkpoint 后再统一写回磁盘

### LRU List、Free List  和 Flush List

缓冲池中对数据使用 LRU List 进行维护，将最频繁访问的数据放在队列头部，而最少使用的数据在尾部。因为在全表搜索的情况下，将最近读取的数据放在头部会影响性能(读入的大部分数据都只用一次)，所以 InnoDB 会将新数据插入 LRU List 的 midpoint 位置(在 LRU List 的 5/8 处，由 innodb_old_blocks_pct 控制)

Free List 用于记录缓存池中的空闲页面，当要将数据从磁盘读到缓冲池时，会先从 Free List 中尝试获取可用的空闲页，再将该页移出 Free List 并加入 LRU List；从 LRU List 淘汰的数据页也会被清空后加入 Free List

Flush List 会记录被修改的数据(脏页)，脏页可以同时存在于 LRU List 和 Flush List，并在 checkpoint 节点写回磁盘(Master Thread / Page Cleaner Thread)

#### 重做日志缓存

InnoDB 会先将 redo log 写入缓存池中的 redo log buffer，再写回磁盘：

- Master Thread 按每秒一次写回磁盘
- 事务提交后
- redo log buffer 容量不足

#### Checkpoint

缓冲池用于平衡 CPU 与磁盘之间的性能差距，但何时将数据从内存持久化到磁盘还需要考虑，InnoDB 使用 checkpoint 来定义一个检查点，表示再次之前的数据修改都被持久化，以解决以下问题：

- redo log 恢复时间长
  - 在 checkpoint 之前的数据都已经被写回磁盘，所以在恢复数据时只需要执行 checkpoint 后的 redo log 即可
- redo log 不可用
  - redo log 是一种环形结构，过期的 redo log 会被新的取代，当 redo log 被重用时必须产生 checkpoint 确保被覆盖的 redo log 不再被需要
- 缓冲池不够用，将脏页写回磁盘
  - 写回磁盘时必须触发 checkpoint

InnoDB 使用 LSN(Log Sequence Number) 来标记 redo log 和 checkpoint 版本，所有小于 LSN checkpoint 的 redo log 都已经写回磁盘

- Sharp Checkpoint：在数据库关闭前进行的将脏页全量写回磁盘，会阻塞主线程
- Fuzzy Checkpoint：部分写回，对系统可用性影响小
  - Master Thread 定期写回
  - 缓冲池不足
  - redo log buffer 空间不足

### Master Thread