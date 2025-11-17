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

Master Thread 具有最高线程优先级，内部由四个循环组成：

- Loop：主循环，分为每秒和每十秒执行的操作，通过 Thread.sleep() 实现，所以时间间隔不一定准确
  - 每秒：
    - 将缓冲池中的日志写回磁盘，即使事务未提交(总是)
    - 合并插入缓存
    - 刷新缓冲池中的脏页
    - 如果当前没有用户活动，切换至 Backgroud Loop
  - 每十秒：
    - 合并插入缓存(总是)
    - 写回日志和脏页(总是)
    - 回收 Undo Page(总是)
- Background Loop：当前没有用户活动或数据库关闭
  - 合并插入缓存
  - 回收 Undo Page
  - 跳回主循环
- Flush Loop：刷新页
- Suspend Loop：挂起主线程，等待事件发生

### 插入缓存

通常行记录的插入顺序是按照主键递增的顺序进行插入的(UUID 等非自增的是随机的)，所以插入聚集索引一般是顺序的，不需要磁盘的随机读取

但一个表还会有多个非聚集的辅助索引，它们的插入不是顺序的(B+ 树的特性)，这时需要通过离散访问非聚集索引页，由此产生的磁盘随机读会影响操作性能

Insert Buffer 就是为了提高非聚集索引插入的性能，当需要插入非聚集索引时，InnoDB 先判断需要插入的非聚集索引页是否在缓冲池中，如果不在则先放入到一个 Insert Buffer 对象中，之后再将 Insert Buffer 和辅助索引页子节点合并

Insert Buffer 的使用需要同时满足：

- 非聚集索引
- 不唯一：插入缓冲时，如果索引唯一，数据库还需要查找索引页来判断插入记录的唯一性，导致离散读的情况发生，这就令避免离散读的 Insert Buffer 失去意义

#### Change Buffer

Insert Buffer 的升级版，对所有修改数据的 DML 都进行缓冲(update insert delete)：

- Insert Buffer
- Delete Buffer：将记录标记删除
- Purge Buffer：真正删除记录

#### Insert Buffer 内部实现

Insert Buffer 的底层结构是一个 B+ 树，它的中间节点存储辅助索引的 key，叶子节点存储辅助索引的值：

- key：mark + space + offset
  - mark：兼容老版本的 Insert Buffer
  - space：辅助索引所在表的表空间 id
  - offset：页所在的偏移量
- value：mark + space + offset + metadata + secondary index record

当一个辅助索引要插入到页时，如果该页不在缓冲池，InnoDB 会先构造一个 search key，再查询 Insert Buffer 的 B+ 树，将记录插入到叶节点中

#### Merge Insert Buffer

插入缓冲与辅助索引页合并的时机：

- 用户查询相关数据，辅助页被读入缓冲区
- 辅助索引页大小不足，避免页分裂造成更大的开销
- 主线程