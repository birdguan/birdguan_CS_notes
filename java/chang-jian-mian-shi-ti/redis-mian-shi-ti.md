# Redis面试题

## **Redis单线程为什么执行速度这么快？**

1. 纯内存操作，避免了大量访问数据库，减少直接读取磁盘数据。Redis将数据存储在内存里面，读写数据的时候不会收到硬盘I/O速度的限制，所以速度更快；
2. 单线程操作：避免了不必要的上下文切换和竞争条件，也不存在多进程或者多线程导致的切换而消耗CPU，不用去考虑各种锁的问题，不存在加锁释放锁操作，没有因为可能出现的死锁而导致的性能消耗
3. 采用了非阻塞I/O多路复用机制

## **Redis数据结构底层实现** 

☞ String

simple dynamic string \(SDS\)数据结构

```text
struct sdshdr{
 //记录buf数组中已使用字节的数量
 //等于 SDS 保存字符串的长度
 int len；
 //记录 buf 数组中未使用字节的数量
 int free；
 //字节数组，用于保存字符串
 char buf[]；
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

优点：

1. 不会出现字符串变更造成的内存溢出问题
2. 获取字符串长度时间复杂度为1
3. 空间预分配，惰性空间释放free字段，会默认留够一定的空间防止多次重分配内存

应用场景：String缓存结构体用户信息，计数

☞ Hash

数组+链表的基础上，进行了一些Hash优化

1. Redis的Hash采用链地址法来处理冲突，没有使用红黑树优化
2. 哈希表节点采用单链表结构
3. rehash优化（采用分而治之的思想，将庞大的迁移工作量划分到每一次CURD中，避免了服务繁忙）

应用场景：保存结构体信息可部分获取不用序列化所有字段

☞ List

List的实现为一个双向链表，既可以支持反向查找和遍历

应用场景：如博客的关注列表，粉丝列表等都可以用Redis的List结构来实现

☞ Set

内部实现是一个value为null的HashMap，实际就是通过计算Hash的方法来快速排序的，这也是set能提供判断一个成员是否在集合内的原因。

应用场景：交集（sinter）、并集（sunion）、差集（sdiff），实现如共同关注、共同喜好、二度好友等功能。

☞ Zset

内部使用HashMap和跳跃表（SkipList）来保证数据的存储和有序，HashMap里放的是成员到score的映射，而跳跃表里存放的是所有的成员，排序依据是HashMap里存的score，使用跳跃表的结构可以获得比较高的查找效率，并在实现上比较简单。跳表：每个节点中维持多个指向其他节点的指针，从而达到快速访问节点的目的。

应用场景：实现延时队列

## **Redis事务**

1. Multi开启事务
2. Exec执行事务块内命令
3. Discard取消事务
4. Watch监视一个或多个key，如果事务执行前key被改动，事务将打断

## **Redis事务的实现特征**

1. 所有命令都会被串行化的顺序执行，事务执行期间，Redis不会再为其他客户端的请求提供任何服务，从而保证了事务中的所有命令被原子地执行
2. Redis事务中如果有一条命令执行失败，其后地命令仍然会被继续执行
3. 在事务开启之前，如果客户端与服务端之间出现通讯故障并导致网络断开，其后所有待执行的语句都不会被服务器执行。然而如果网络中断事件是发生客户端执行EXEC命令之后，那么该事务中的所有命令都会被服务器执行
4. 当使用Append-Only模式时，Redis会通过调用系统函数write将事务内的所有写操作在本次调用中全部写入磁盘；然而如果在写入的过程中出现系统崩溃，如电源故障导致的宕机，那么此时也许只有部分数据被写入到磁盘，而另一部分数据却已经丢失。Redis服务器会在重新启动时执行一系列必要的一致性检测，一旦发现类似问题，就会立即退出并给出相应的错误提示。此时，我们要充分利用Redis工具包中提供的redis-check-aof工具，该工具可以帮助我们定位到数据不一致的错误，并将已经写入的部分数据进行回滚。修复之后我们就可以再次重启Redis服务器了。

## **Redis同步机制**

☞ 全量拷贝

1. slave第一次启动时，连接Master，发送PSYNC命令
2. master会执行bgsave命令来生成rdb文件，期间的所有写命令将被写入缓冲区
3. master bgsave执行完毕，向slave发送rdb文件
4. slave收到rdb文件，丢弃所有旧数据，开始载入rdb文件
5. rdb文件同步结束后，slave执行从master缓冲区发送过来的所有写命令
6. 此后master没执行一个写命令，就向slave发送相同的写命令

☞ 增量拷贝

如果出现网络闪断或者命令丢失等异常情况，从节点之前保存了自身已复制的偏移量和主节点的运行ID，主节点根据偏移量把复制积压缓存里的数据发送给从节点，保证主从复制进入正常状态。

## **Redis集群模式性能优化**

1. Master最好不要做任何持久化工作，如RDB内存快照和AOF日志文件
2. 如果数据比较重要，某个slave开启AOF备份数据，策略设置为每秒同步一次
3. 为了主从复制的速度和连接的稳定性，Master和Slave最好在同一个局域网内
4. 尽量避免在压力很大的主库上增加从库
5. 主从复制不要用图状结构，用单向链表结构更为稳定，即：Master &lt;- Slave1 &lt;- Slave2 &lt;- Slave3...这样的结构方便解决单点故障问题，实现Slave对Master的替换。如果Master挂了，可以立即启用Slave1做Master，其他不变。

## **Redis集群方案**

1. 官方cluster方案
2. twemproxy
3. codis

## **集群不可用场景**

1. Master挂掉，且当前master没有slave
2. 集群超过半数以上Master挂掉，无论是否有Slave集群进入Fail状态

## **Redis最适合的场景**

1. 会话缓存session cache
2. 排行榜/计数器ZRANGE
3. 发布/订阅

## **缓存淘汰策略**

1. 先进先出算法（FIFO）
2. 最近最少使用（Least Frequently Used, LFU）
3. 最长时间未被使用（Least Recently Used, LRU）

当存在热点数据时，LRU的效率很好，但偶发性的、周期性的批量操作会导致LRU命中率急剧下降，缓存污染情况比较严重

## **缓存雪崩以及处理方法**

☞ 产生原因

同一时刻大量缓存失效

☞ 处理方法

1. 缓存数据增加过期标记
2. 设置不同的缓存失效时间
3. 双层缓存策略，C1为短期，C2为长期
4. 定时更新策略

## **缓存击穿原因以及处理办法**

☞ 产生原因

频繁请求查询系统中不存在的数据导致

☞ 处理方法

1. cache null策略，查询反馈结果为null仍然缓存这个null结果，设置不超过5分钟的过期时间
2. 布隆过滤器，所有可能存在的数据映射到足够大的bitmap中
   1. google过滤器过滤器：基于内存，重启失效不支持大数据量，无法应用在分布式场景
   2. redis布隆过滤器：可扩展性，不存在重启失效问题，需要网络io，性能低于google布隆过滤器

## **Redis阻塞原因**

1. 数据结构使用不合理bigkey
2. CPU饱和
3. 持久化阻塞，rgb fork子线程，aof每秒刷盘等 

## **hot key出现造成集群访问量倾斜解决办法**

1. 使用本地缓存
2. 利用分片算法的特性，对key进行打散处理（给hot key加上前缀后者后缀，把一个hotkey的数量变成redis实例个数N的倍数M，从而由访问一个redis key变成访问N\*M个redis key）

## **Redis如何做持久化**

bgsave做镜像全量持久化，aof做增量持久化。因为bgsave会耗费较长时间，不够实时，在停机的时候会导致大量丢失数据，所以需要aof来配合使用。在redis实例重启时，会使用bgsave持久化文件重新构建内存，再使用aof重放近期的操作指令来实现完整重启之前的状态。

## **bgsave的原理是什么？**

fork和cow。fork是指redis通过创建子线程来进行bgsave操作，cow是指copy on write，子线程创建后，父子线程共享数据段，父线程继续提供读写服务，写进的页面数据会逐渐和子线程分离开来。

## **RDB和AOF区别**

1. RDB文件格式紧凑，方便数据恢复，保存rdb文件时父线程会fork出子线程由其完成具体持久化操作，最大化redis性能，恢复大数据集速度更快，只有手动提交save命令或关闭命令时才触发备份操作；
2. AOF记录服务器的每次写操作（默认1s写入一次），保存数据更完整，在redis重启时会重放这些命令来恢复数据，操作效率高，故障丢失数据更少，但是文件体积更大

## **1亿个key，其中有10w个key是以某个固定的已知的前缀开头的，如何将它们全部找出来？**

* 使用keys指令可以扫出指定模式的key列表，如果这个redis正在给线上的业务提供服务，那使用keys指令会存在问题：因为redis是单线程的，keys指令会导致线程阻塞一段时间，线上服务会停顿，直到指令执行完毕，服务才能恢复。
* 这个时候可以使用scan指令，scan指令可以无阻塞地提取出指定模式的key列表，但是会有一定的重复概率，在客户端做一次去重就可以了，但是整体花费的时间会比直接用keys指令长。

## **如何使用Redis做异步队列？**

一般使用List结构最为队列，rpush生产消息，lpop消费消息。当lpop没有消息的时候，要适当sleep一会儿再试；List还有个指令叫blpop，在没有消息的时候，它会阻塞直到消息到来。

## **Redis如何实现延时队列？**

使用sortedset，想要执行时间的时间戳作为score，消息内容作为key调用zadd来生产消息，消费者用zrangebyscore指令获取N秒之前的数据轮询进行处理。

## **为啥redis zset使用跳跃链表而不是红黑树实现？**

1. skiplist的复杂度和红黑树一样，而且实现起来更简单
2. 在并发环境下红黑树在插入和删除时需要rebalance，性能不如跳表

