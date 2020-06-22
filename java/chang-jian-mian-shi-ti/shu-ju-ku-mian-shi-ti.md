# 数据库面试题

## **数据库三范式**

1. 确保每列的原子性，即列的信息，不能再分解
2. 非主键列不存在对主键的部分依赖（要求每个表只描述一件事情）
3. 满足第二范式，并且表中的列不存在对非主键列的传递依赖

## **数据库主从复制原理** 

1. 主库db的更新事件（update, insert, delete）被写到binlog
2. 主库创建一个binlog dump thread线程，把binlog的内容发送到从库
3. 从库创建一个I/O线程，读取主库传过来的binlog内容并写入到relay log
4. 从库还会创建一个SQL线程，从relay log里面读取内容写入到slave的db

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMTExNDUxMi02MDIyMzZkZTYxYjFmMmQ5LnBuZw?x-oss-process=image/format,png)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)​

## **复制方式分类**

1. 异步复制（默认）：主库写入binlog日志后即可成功返回客户端，无需等待binlog传递给从库的过程，但是一旦出现主库宕机，就有可能出现丢失数据的情况。
2. 半同步复制（5.5版本后，安装半同步复制插件）：确保从库接受完成从主库传递火来的binlog内容已经写入到自己的relay log（床送log）后才通知从库上面的等待线程。如果等待超时，则关闭半同步复制，并自动转换为异步复制模式，直到至少有一台从库通知主库已经接收到binlog信息为止。

## **存储引擎**

1. MyISAM：不支持数据库事务，行级锁，外键；插入更新需锁表，效率低，查询速度快；使用非聚簇索引。
2. InnoDB：支持事务，底层为B+树实现，适合处理多重并发更新操作，普通select都是快照读，快照读不加锁；使用聚簇索引。

## **聚簇索引与非聚簇索引**

☞ 聚簇索引

聚簇索引就是按照每张表的主键构造一颗B+树，同时叶子节点中存放的就是整张表的行记录数据，也将蹴鞠索引的叶子节点称为数据页。这个特性决定了索引组织表中数据也是索引的一部分，每张表只能拥有一个聚簇索引。

InnoDB通过主键聚集数据，如果没有定义主键，InnoDB会选择非空的唯一索引代替。如果没有这样的索引，InnoDB会隐式地定义一个主键row\_id来作为聚簇索引。

优点：

1. 数据访问更快，因为聚簇索引将索引和数据保存在同一个B+树中，因此从聚簇索引中获取数据比非聚簇索引更快；
2. 聚簇索引对于主键的排序查找和范围查找速度非常快

缺点：

1. 插入速度严重依赖于插入顺序，按照主键的顺序插入是最快的方式，否则会出现页分裂，严重影响性能。因此，对于InnoDB表，一般定义自增的ID列作为主键；
2. 更新主键的代价很高，因为将导致被更新的行移动。因此，对于InnoDB表，一般定义主键不可更新；
3. 二级索引访问需要两次索引查找，第一次找到主键值，第二次根据主键值找到行数据

☞ 非聚簇索引（辅助索引）

在聚簇索引质上创建得索引称之为辅助索引，辅助索引访问数据总是需要二次查找。辅助索引叶子节点存储得不再是行得物理位置，而是主键值。通过辅助索引首先找到得是主键值，再通过主键索引找到数据行的数据页，再通过数据页中的Page Directory找到数据行。

InnoDB辅助索引的叶子节点并不包含行记录的全部数据，叶子节点除了包含键值外，还包含了相应行数据的聚簇索引键。

辅助索引的存在不影响数据在聚簇索引中的组织，所以一张表可以有多个辅助索引。在InnoDB中有时也称辅助索引为二级索引。

## **使用聚簇索引为什么查询速度会更快？**

使用聚簇索引找到包含第一个值的行后，便可以确保包含后续索引值的行物理相邻。

## **建立聚簇索引有什么需要注意的地方吗？**

在聚簇索引中不要包含经常修改的列，因为码值修改后，数据行必须以到新的位置，所以此时会重排，造成很大的资源浪费。

## **InnoDB表对主键生成策略是什么样的？**

优先使用用户自定义主键作为主键，如果用户没有定义主键，则选取一个unique键作为主键，如果表中连unique键都没有定义的话，则InnoDB会为表默认添加一个名为row\_id隐藏列作为主键。

## **非聚簇索引最多可以有多少个？**

每个表最多可以建立249个非聚簇索引。非聚簇索引需要大量的硬盘空间和内存。

## **BTree和HASH索引有什么区别？**

1. BTree索引可能需要多次运用折半查找来找到对应的数据块；HASH索引是通过HASH函数，计算出HASH值，在表中找出对应的数据
2. 大量不同数据等值精确查找，HASH索引效率通常比BTree高；HASH索引不支持模糊查询、范围查询和联合索引中的最左匹配原则，而这些BTree都支持。

## **数据库索引优缺点**

1. 需要查询、排序、分组和联合操作的字段适合建立索引
2. 索引多，数据表更新慢，尽量使用字段值不重复比例大的字段作为索引，联合索引比多个独立索引效率高
3. 对数据进行频繁查询建立索引，如果要频繁更改数据不建议使用索引
4. 当对表中的数据进行增加、删除和修改时，索引也要动态地维护，降低数据地维护速度

## **索引的底层实现是B+ Tree，为何不用红黑树，B树？**

1. B+ Tree非叶子节点指存储键值信息，降低B+ Tree的高度，所有叶子节点之间都有一个链指针，数据记录存放在叶子节点中
2. 红黑树这种结构，h明显深得多，效率明显比B-Tree差很多
3. B+ Tree也存在劣势，由于键会重复出现，因此会占用更多的空间。但是与带来的性能优势相比，空间劣势往往可以接受，因此B+ Tree再数据库中比B树应用更加广泛

## **索引失效条件**

1. 条件是or，如果还想让or条件生效，给or每个字段加索引
2. like开头%
3. 如果列类型是字符串，那一定要在条件中将数据使用引号引用起来，否则不会使用索引
4. where中索引列使用了函数或者有运算

## **数据库事务特点**

ACID 原子性，一致性，隔离性，永久性

☞ 数据库事务是如何实现的？

1. 通过预写日志方式实现的，redo和undo机制是数据库实现事务的基础
2. redo日志用来在断电/数据库崩溃等状况发生时重演一次刷数据的过程，把redo日志里的数据刷到数据库里，保证事务的持久性
3. undo日志是在事务执行失败的时候撤销对数据库的操作，保证了事务的原子性

☞ 数据库事务隔离级别

1. READ UNCOMMITTED（读未提交数据）：允许事务读取未被其他事务提交的变更数据，会出现脏读、不可重复读和幻读问题。
2. READ COMMITTED（读已提交数据）：只允许事务读取已经被其他事务提交的变更数据，可避免脏读，仍会出现不可重复读和幻读问题。
3. REAPABLE READ（可重复读）：确保事务可以多次从一个字段中读取相同的值，在此事务持续期间，禁止其他事务对此字段的更新，可以避免脏读和不可重复读，仍会出现幻读问题。
4. SERIALIZABLE（序列化）：确保事务可以从一个表中读取相同的行，在这个事务持续期间，禁止其他事物对该表执行插入、更新和删除操作，可避免所有并发问题，但性能非常低。

☞ 脏读、不可重复读和幻读

* 脏读：事务A对这个数据进行了修改，而这种修改还没有提交到数据库中，这时，另外一个事务B也访问这个数据，然后使用了这个数据。
* 不可重复读：在事务1内，读取了1个数据，事务1还没有结束，事务2也访问了这个数据，修改了这个数据并提交。紧接着，事务1又读取这个数据，由于事务2的修改，事务1两次读到的数据可能是不一样的。
* 幻读：当某个事物在读取某个范围内的记录时，另外一个事物又在该范围内插入了新的记录，当之前的事物再次读取该范围的记录时，会产生幻行。InnoDB存储引擎通过多版本并发控制（MVCC）解决了幻读的问题。

不可重复读和幻读的区别是：前者是指读到了已经提交了的事物的更改数据（修改或删除），后者是读到了其他已提交事物的新增数据。

对于这两种问题解决采用不同的方法，防止读到更改数据，只需要对操作的数据添加行级锁，防止操作中的数据发生变化；而防止读到新增数据，往往需要添加表级锁，将整张表锁定，防止新增数据。

## **七种事务传播行为**

1. Propagation.REQURED（默认）如果当前存在事物，则加入该事务，如果当前不存在事务，则创建一个新的事务
2. Propagation.SUPPORTS 如果当前存在事务，则加入该事务；如果当前不存在事务，则以非事务的方式继续运行
3. Propagation.MANDATORY 如果当前存在事务，则加入该事务；如果当前不存在事务，则抛出异常
4. Propagation.REQUIRED\_NEW 重新创建一个新的事务，如果当前存在事务，延缓当前的事务
5. Propagation.NOT\_SUPPORTED 以非事务的方式运行，如果当前存在事务，暂停当前的事务
6. Propagation.NEVER 以非事物的方式运行，如果当前存在事物，则抛出异常
7. Propagation.NESTED 如果没有，就新建一个事物；如果有，就在当前事务中嵌套其他事务

## **产生死锁的四个必要条件**

1.  互斥：资源x在任意时候只能被一个线程持有
2. 占有且等待：线程1占有资源x的同时等待资源y，并不释放x
3. 不可抢占：资源x一旦被线程1抢占，其他线程不能抢占x
4. 循环等待：线程1持有x，等待y，线程2持有y，等待x

当全部满足时才会产生死锁。

## **@Transcation**

底层实现是AOP，动态代理

* 通过Spring代理实现。生成当前类的代理类，调用代理类的invoke\(\)方法，在invoke\(\)方法中调用TransactionInterceptor拦截器的invoke\(\)方法；
* 非public方式其事务是失效的；
* 自调用也会失效，因为动态代理机制导致；
* 多个方法外层加入try...catch，解决办法是可以在catch里throw new RuntimeException\(\)来处理
