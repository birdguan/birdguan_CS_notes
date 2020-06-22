# 数据库系统原理

## 一、事务

### 概念 <a id="%E6%A6%82%E5%BF%B5"></a>

事务指的是满足ACID特性的一组操作，可以通过Commit提交一个事务，也可以使用Rollback进行回滚。

![](https://img-blog.csdnimg.cn/20200304135525512.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMwMjQxNzA5,size_16,color_FFFFFF,t_70)

### ACID <a id="ACID"></a>

1. 原子性（Atomivity）

事务被视为不可分割的最小单元，事务的所有操作要么全部提交成功，要么全部提交失败回滚。

回滚可以用回滚日志（Undo Log）来实现，回滚日志记录着事务所执行的修改操作。

2.一致性（Consistency）

数据库在事务执行前后都保持一致性状态。在一致性状态下，所有事务**对同一个数据的读结果都是相同的**。

3.隔离性（Isolation）

一个事务所做的修改在最终提交以前，对其它事务是不可见的。

4.持久性（Durability）

一旦事务提交，则其所作的修改会永远保存到数据库中。即使系统发生崩溃，事务执行的结果也不能丢失。

系统发生崩溃可以用重做日志（Redo Log）进行恢复，从而实现持久性。与回滚日志记录数据的逻辑修改不同，重做日志记录的是数据页的物理修改。

对事务的ACID的理解：

* 只有满足一致性，事务的执行结果才是正确的。
* 在无并发的情况下，事务串行执行，隔离性一定能够满足。此时只要满足原子性，就一定满足一致性。
* 在并发的情况下，多个事务并行执行，事务不仅要满足原子性，还需要满足隔离性，才能满足一致性。
* 事务满足持久化是为了能应对系统崩溃的情况。

### AUTOCOMMIT <a id="AUTOCOMMIT"></a>

MySQL默认采用自动提交模式。也就是说，如果不显式使用START TRANSCATION语句来开始一个事务，那么每个查询操作都会被当做一个事务并自动提交。

## 二、并发一致性问题 <a id="%E4%BA%8C%E3%80%81%E5%B9%B6%E5%8F%91%E4%B8%80%E8%87%B4%E6%80%A7%E9%97%AE%E9%A2%98"></a>

### 丢失修改 <a id="%E4%B8%A2%E5%A4%B1%E6%95%B0%E6%8D%AE"></a>

T1和T2两个事务都对一个数据进行修改，T1先修改，T2后修改，T2的修改覆盖了T1的修改。

![](https://img-blog.csdnimg.cn/20200304141016613.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMwMjQxNzA5,size_16,color_FFFFFF,t_70)

### 读脏数据 <a id="%E8%AF%BB%E8%84%8F%E6%95%B0%E6%8D%AE"></a>

T1修改数据后回滚，T2读到的T1回滚中的修改数据。

![](https://img-blog.csdnimg.cn/20200304141156697.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMwMjQxNzA5,size_16,color_FFFFFF,t_70)

### 不可重复读 <a id="%E4%B8%8D%E5%8F%AF%E9%87%8D%E5%A4%8D%E8%AF%BB"></a>

T2读取一个数据，T1对该数据做了修改，T2再次读取该数据，两次读取的结果不相同。

![](https://img-blog.csdnimg.cn/20200304141311818.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMwMjQxNzA5,size_16,color_FFFFFF,t_70)

### 幻影读 <a id="%E5%B9%BB%E5%BD%B1%E8%AF%BB"></a>

T1读取某个范围的数据，T2在这个范围内插入新的数据，T1再次读取这个范围的数据，此时两次读取的结果不一致。

![](https://img-blog.csdnimg.cn/20200304141422402.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMwMjQxNzA5,size_16,color_FFFFFF,t_70)

* 不可重复度和幻影读的区别

借用[该博客](https://www.jianshu.com/p/4b6e0103a13c)观点，避免不可重复读只需要锁住满足条件的记录（即锁行），避免幻影读需要所著满足条件及其相近的记录（即锁表）

产生并发不一致性问题的主要原因是破坏了事务的隔离性，解决方法是通过并发控制来保证隔离性。并发控制可以通过封锁来实现，但是封锁操作需要用户自己控制，相当复杂。数据库管理系统提供了事务的隔离级别，让用户以一种更轻松的方式处理并发一致性问题。

## 三、封锁 <a id="%E4%B8%89%E3%80%81%E5%B0%81%E9%94%81"></a>

### 封锁粒度 <a id="%E5%B0%81%E9%94%81%E7%B2%92%E5%BA%A6"></a>

MySQL中提供了两种封锁粒度：行级锁和表级锁

应该尽量只锁定需要修改的那部分数据，而不是所有的资源。锁定的数据量越少，发生锁争用的可能就越小，系统的并发程度就越高。

加锁需要消耗资源，锁的各种操作（包括获取锁、释放锁、以及检查锁状态）都会增加系统开销。因此封锁粒度越小，系统开销越大。

在选择封锁粒度时，需要在琐开销和并发成都之间做一个权衡。

### 封锁类型 <a id="%E5%B0%81%E9%94%81%E7%B1%BB%E5%9E%8B"></a>

1.读写锁

* 互斥锁（Exclusive），X锁，又称写锁
* 共享锁（Shared），S锁，又称读锁

关于X锁和S锁的规定：

* 一个事务对数据对象A加了X锁，就可以对A进行读取和更新。加锁期间其他事务不能对A加任何锁。
* 一个事务对数据对象A加了S锁，可以对A进行读取操作，但是不能进行更新操作。加锁期间其他事务能对A加S锁，但是不能加X锁。

X锁和S锁的兼容关系图：

![](https://img-blog.csdnimg.cn/2020030414363329.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMwMjQxNzA5,size_16,color_FFFFFF,t_70)

2.意向锁

使用意向锁可以更容易地支持多粒度封锁。

在存在行级锁和表级锁地情况下，事务T想要对表A加X锁，就需要先检测是否有其它事务对表A或者表A中的任意一行加了锁，那么就需要对表A的每一行都检测一次，这是十分耗时的。

意向锁在原来的X/S锁之上引入了IX/IS，IX/IS锁都是表锁，用来表示一个事务想要在表中的某个数据行上加上X锁或S锁。有以下两个规定：

* 一个事务在获得某个数据行对象的S锁之前，必须先获得表的IS锁或者更强的锁；
* 一个事务在获得某个数据行对象的X锁之前，必须获得表的IX锁。

例如，事务T想要对表A加X锁，只需要先检测是否有其他事务对表A加了X/IX/S/IS锁，如果加了就表示有其他事务正在使用这个表或表中的某一行的锁，因此事务T加X锁失败。

各种锁的兼容关系：

![](https://img-blog.csdnimg.cn/20200304145156168.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMwMjQxNzA5,size_16,color_FFFFFF,t_70)

* 任意的IS/IX锁之间都是兼容的，因为它们表示想要对表加锁，而不是真正加锁；
* 表级的IX锁和行级的X锁兼容，两个事务可以对两个数据行加X锁

### 封锁协议 <a id="%E5%B0%81%E9%94%81%E5%8D%8F%E8%AE%AE"></a>

1. 三级封锁协议

* 一级封锁协议
  * 事务要修改数据A时必须加X锁，直到T结束时才释放锁。
  * 可以解决丢失修改问题，因为不能同时有两个事务对一个数据进行修改，那么事务的修改就不会被覆盖。

![](../../.gitbook/assets/image%20%2867%29.png)

* 二级协议锁
  * 在一级协议锁的基础上，要求读数据A时必须加S锁，读取完马上释放S锁
  * 可以解决读脏数据的问题，因为如果一个事务对数据A进行修改，根据1级封锁协议，会加X锁，那么就不能再加S锁了，也就是不会读入数据。

![](../../.gitbook/assets/image%20%2818%29.png)

* 三级协议锁
  * 在二级的基础上，要求读数据A时必须加S锁，直到事务结束了才能释放S锁。
  * 可以解决不可重复读的问题，因为读A时，其它事务不能对A加X锁，从而避免了在读的期间数据发生改变。

![](../../.gitbook/assets/image%20%2835%29.png)

2.两段锁协议

加锁和解锁分为两个阶段进行。

可串行化调度是指，通过并发控制，使得并发执行的事务结果与某个穿行执行的事务结果相同。串行执行的事务互不干扰，不会出现并发一致性问题。

事务遵循两段锁协议是保证可串行化调度的充分条件。例如以下操作满足两段锁协议，它是可串行化调度：

lock-x\(A\)...lock-s\(B\)...lock-s\(C\)..unlock\(A\)...unlock\(C\)...unlock\(B\)

但不是必要条件，例如以下操作不满足两段锁协议，但它还是可串行化调度。

lock-x\(A\)...unlock\(A\)...lock-s\(B\)...unlock\(B\)...lock-s\(C\)...unlock\(C\)

### MySQL隐式与显示锁定 <a id="MySQL%E9%9A%90%E5%BC%8F%E4%B8%8E%E6%98%BE%E7%A4%BA%E9%94%81%E5%AE%9A"></a>

MySQL的InnoDB存储引擎采用两段锁协议，会根据隔离级别在需要的时候自动加锁，并且所有的锁都是在同一时刻被释放，这被称为隐式锁定。

InnoDB也可以使用特定的语句进行显式锁定：

```text
SELECT ... LOCK In SHARE MODE;
SELECT ... FOR UPDATE;
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

## 四、隔离级别 <a id="%E5%9B%9B%E3%80%81%E9%9A%94%E7%A6%BB%E7%BA%A7%E5%88%AB"></a>

### 未提交读 <a id="%E6%9C%AA%E6%8F%90%E4%BA%A4%E8%AF%BB"></a>

事务中的修改，即使没有提交，对其它事务也是可见的。

### 提交读 <a id="%E6%8F%90%E4%BA%A4%E8%AF%BB"></a>

一个事务只能读取已经提交的事务所做的修改。换句话说，一个事务所做的修改在提交之前对其它事务是不可见的。

### 可重复读 <a id="%E5%8F%AF%E9%87%8D%E5%A4%8D%E8%AF%BB"></a>

保证在同一个事务中多次读取同一数据的结果是一样的。

### 可串行化 <a id="%E5%8F%AF%E4%B8%B2%E8%A1%8C%E5%8C%96"></a>

强制事务串行执行，这样多个事务互补干扰，不会出现并发一致性问题。

该隔离级别需要加锁实现，因为要使用加锁机制保证同一时间只有一个事务执行，也就是保证事务串行执行。

![](https://img-blog.csdnimg.cn/20200304151930585.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMwMjQxNzA5,size_16,color_FFFFFF,t_70)

## 五、多版本并发控制 <a id="%E4%BA%94%E3%80%81%E5%A4%9A%E7%89%88%E6%9C%AC%E5%B9%B6%E5%8F%91%E6%8E%A7%E5%88%B6"></a>

多版本并发控制（Multi-Version Concurrency Control, MVCC）是MySQL的InnoDB存储引擎实现隔离级别的一种具体方式，用于实现提交读和可重复读这两种隔离级别。而未提交读隔离级别总是读取最新的数据行，要求很低，无需使用MVCC。可串行化隔离级别需要对所有读取的行加锁，单纯使用MVCC无法实现。

### 基本思想 <a id="%E5%9F%BA%E6%9C%AC%E6%80%9D%E6%83%B3"></a>

加锁能解决多个事务同时执行时出现的并发一致性问题。在实际场景中读操作往往多于写操作，因此又引入了读写锁来避免不必要的加锁操作，例如读和读没有互斥关系。读写锁中读和写操作仍然是互斥的，而MVCC利用了多版本思想，写操作更新最新的版本快照，而读操作去读旧版本快照，没有互斥关系，这一点和CopyOnWrite类似。

在MVCC中事务的修改操作（DELETE, INSERT, UPDATE）会为数据行新增一个版本快照。

脏读和不可重复读最根本的原因是事务读取到其它事务未提交的修改。在事务进行读取操作时，为了解决脏读和不可重复读问题，MVCC规定只能读取已经提交的快照。当然一个事务可以读取自身未提交的快照，这不算脏读。

### 版本号 <a id="%E7%89%88%E6%9C%AC%E5%8F%B7"></a>

系统版本号SYS\_ID：是一个递增的数字，每开始一个新的事务，系统版本号就会自动递增。

事务版本号TRX\_ID：事务开始时的系统版本号。

### Undo日志 <a id="Undo%E6%97%A5%E5%BF%97"></a>

MVCC的多版本指的是多个版本的快照，快照存储在Undo日志中，该日志通过回滚指针ROLL\_PTR把一个行所有的快照连接起来。

例如在MySQL创建一个表t，包含主键id和一个字段x。我们先插入一个数据行，然后对该数据行执行两次更新操作。

```text
INSERT INTO t(id, x) VALUES (1, "a");
UPDATE t SET x="b" WHERE id=1;
UPDATE t SET x="c" WHERE id=1;
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

 因为没有使用START TRANSACTION将上面的操作当成一个事务来执行，根据MySQL的AUTOCOMMIT机制，每个操作都会被当成一个事务来执行，所以上面的操作总共设计到三个事务。快照中除了记录事务版本号TRX\_ID和操作之外，还记录了一个bit的DEL字段，用于标记是否被删除。

![](https://img-blog.csdnimg.cn/2020030417210375.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMwMjQxNzA5,size_16,color_FFFFFF,t_70)

INSERT, UPDATE, DELETE操作会创建一个日志，并将事务版本号TRX\_ID写入。DELETE可以看成是一个特殊的UPDATE，还会额外将DEL字段设置为1.

### ReadView <a id="ReadView"></a>

MVCC维护了一个ReadView结构，主要包含了当前系统未提交的事务列表TRX\_IDs{TRX\_ID\_1, TRX\_ID\_2, ...}，还有该列表的最小值TRX\_ID\_MIN和最大值TRX\_ID\_MAX。

![](https://img-blog.csdnimg.cn/20200304173155161.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMwMjQxNzA5,size_16,color_FFFFFF,t_70)

在进行SELECT操作时，根据数据快照的TRX\_ID与TRX\_ID\_MIN和TRX\_MAX之间的关系，从而判断数据行快照是否可以使用：

* TRX\_ID &lt; TRX\_ID\_MIN，表示该数据行快照是在所有未提交事务之前进行更改的，因此可以使用。
* TRX\_ID &gt; TRX\_ID\_MAX，表示该数据行快照是在事务启动之后被更改的，因此不可使用。
* TRX\_ID\_MIN &lt;= TRX\_ID &lt;= TRX\_ID\_MAX，需要根据隔离级别再进行判断：
  * 提交读：如果TRX\_ID在TRX\_IDs列表中，表示该数据行快照对应的事务还未提交，则该快照不可使用。否则表示已经提交，可以使用。
  * 可重复读：都不可以使用。因为如果可以使用的话，那么其它事务也可以读到这个数据行快照并进行修改，那么当前事务再去读这个数据行得到的值就会发生改变，也就是出现了不可重复读的问题。

在数据行快照不可使用的情况下，需要沿着Undo Log回滚指针ROLL\_PTR找到下一个快照，再进行上面的判断。

### 快照读与当前读 <a id="%E5%BF%AB%E7%85%A7%E8%AF%BB%E4%B8%8E%E5%BD%93%E5%89%8D%E8%AF%BB"></a>

1.快照读

MVCC的SELECT操作是快照中的数据，不需要进行加锁操作。

```text
SELECT * FROM table ...;
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

 2.当前读

MVCC会对其它数据库操作（INSERT, UPDATE, DELETE）进行操作操作，从而读取最新的数据。可以看到MVCC并不是完全不用加锁，而只是避免了SELECT的加锁操作。

不过在进行SELECT操作时，可以强制指定进行加锁操作。以下第一个语句需要加S锁，第二个需要加X锁。

```text
SELECT * FROM table WHERE ? lock in share mode;
SELECT * FROM table WHERE ? for update;
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

## 六、Next-Key Locks <a id="%E5%85%AD%E3%80%81Next-Key%20Locks"></a>

Next-Key Locks是MySQL的InnoDB存储引擎的一种加锁实现。

MVCC不能解决幻读问题，Next-Key Locks就是为了解决这个问题而存在的。在可重复读（REPEATABLE READ）隔离级别下，使用MVCC+Next-Key Locks可以解决幻读问题。

### Record Locks <a id="Record%20Locks"></a>

锁定一个记录上的索引，而不是记录本身。

如果表没有设置索引，InnoDB会自动在主键创建隐藏的聚簇索引，因此Record Locks仍然可以使用。

### Gap Locks <a id="Gap%20Locks"></a>

锁定索引之间的间隙，但是不包含索引本身。例如当一个事务执行以下语句，其它事务就不能在t.c索引为15中插入数据。

```text
SELECT c FROM t WHERE c BETWEEN 10 and 20 FOR UPDATE;
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

### Next-Key Locks <a id="Next-Key%20Locks"></a>

它是Record Locks和Gap Locks的结合，不仅锁定一个记录上的索引，也锁定索引之间的间隙。它锁定的是一个前开后闭区间。

## 七、关系数据库设计理论 <a id="%E4%B8%83%E3%80%81%E5%85%B3%E7%B3%BB%E6%95%B0%E6%8D%AE%E5%BA%93%E8%AE%BE%E8%AE%A1%E7%90%86%E8%AE%BA"></a>

### 函数依赖 <a id="%E5%87%BD%E6%95%B0%E4%BE%9D%E8%B5%96"></a>

记A-&gt;B表示A函数决定B，也可以说B函数依赖于A。

如果{A1, A2, ..., An}是关系的一个或多个属性的集合，该集合函数决定了关系的其它所有属性并且是最小的，那么就称该集合为键码。

对于A-&gt;B，如果能找到A的真子集A'，使得A'-&gt;B，那么A-&gt;B就是部分函数依赖，否则就是完全函数依赖。

对于A-&gt;B，B-&gt;C，则A-&gt;C是一个传递函数依赖。

### 异常 <a id="%E5%BC%82%E5%B8%B8"></a>

举例说明：

![](https://img-blog.csdnimg.cn/20200304181349572.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMwMjQxNzA5,size_16,color_FFFFFF,t_70)

 不符合范式的关系，会产生很多异常，主要有以下四种异常：

* 冗余数据：例如学生-2出现了两次
* 修改异常：修改了一个记录中的信息，但是另一个记录中相同的信息却没有被修改
* 删除异常：删除一个信息，那么也会丢失其它信息。例如删除了课程-1需要删除第一行和第三行，那么学生-1的信息就会丢失。
* 插入异常：例如想要插入一个学生的信息，如果这个学生还没选课，那么就无法插入。

### 范式 <a id="%E8%8C%83%E5%BC%8F"></a>

范式理论是为了解决以上提到的四种异常。

高级别范式依赖于低级别范式，1NF是最低级别的范式。

1. 第一范式

属性不可分

2.第二范式

每个非主属性完全函数依赖于键码。

可以通过分解来满足。

分解前：

![](https://img-blog.csdnimg.cn/20200304183658231.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMwMjQxNzA5,size_16,color_FFFFFF,t_70)

以上学生课程关系中，{Sno, Cname}为键码，有如下函数依赖：

* Sno-&gt;Sname, Sdept
* Sdept-&gt;Mname
* Sno, Cname -&gt; Grade

Grade完全函数依赖于键码，它没有任何冗余数据，每个学生的每门课都有特定的成绩。

Sname, Sdept和Mname都部分依赖于键码，当一个学生选修了多门课时，这些数据就会出现多次，造成大量冗余数据。

分解后：

![](https://img-blog.csdnimg.cn/2020030419430757.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMwMjQxNzA5,size_16,color_FFFFFF,t_70)

![](https://img-blog.csdnimg.cn/20200304194321928.png)

3. 第三范式

非主属性不传递函数依赖于键码。

上面的关系-1中存在以下传递函数依赖：

* Sno -&gt; Sdept -&gt; Mname

可以进行以下分解：

![](https://img-blog.csdnimg.cn/20200304194536665.png)

![](https://img-blog.csdnimg.cn/20200304194550547.png)

## 八、ER图 <a id="%E5%85%AB%E3%80%81ER%E5%9B%BE"></a>

Entity-Relationship，有三个组成部分：实体、属性、联系

用来进行关系型数据库系统的概念设计。

### 实体的三种联系 <a id="%E5%AE%9E%E4%BD%93%E7%9A%84%E4%B8%89%E7%A7%8D%E8%81%94%E7%B3%BB"></a>

包含一对一、一对多，多对多三种。

* 如果A到B是一对多关系，那么画个带箭头的线段指向B；
* 如果是一对一，画两个带箭头的线段；
* 如果是多对多，画两个不带箭头的线段。

下图的Course和Student是一对多的关系。

![](https://img-blog.csdnimg.cn/20200304195054541.png)

### 表示出现多次的关系 <a id="%E8%A1%A8%E7%A4%BA%E5%87%BA%E7%8E%B0%E5%A4%9A%E6%AC%A1%E7%9A%84%E5%85%B3%E7%B3%BB"></a>

一个实体在联系出现多次，就要用几条线连接。

下图表示一个课程的先修关系，先修关系出现两个Course实体，第一个是先修课程，后一个是后修课程，因此需要用两条线来表示这种关系。

![](https://img-blog.csdnimg.cn/20200304195402123.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMwMjQxNzA5,size_16,color_FFFFFF,t_70)

### 联系的多向性 <a id="%E8%81%94%E7%B3%BB%E7%9A%84%E5%A4%9A%E5%90%91%E6%80%A7"></a>

虽然老师可以开设多门课程，并且可以教授多名学生，但是对于特定的学生和课程，只有一个老师教授，这就构成了一个三元联系。

![](https://img-blog.csdnimg.cn/20200304195547937.png)

### 表示子类 <a id="%E8%A1%A8%E7%A4%BA%E5%AD%90%E7%B1%BB"></a>

用一个三角形和两条线来连接类和子类，与子类有关的属性和联系都连接到子类，而与父类和子类都有关的连接到父类上。

![](https://img-blog.csdnimg.cn/20200304195712381.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMwMjQxNzA5,size_16,color_FFFFFF,t_70)

注：本文为[Cyc2018](https://github.com/CyC2018/CS-Notes/blob/master/notes/%E6%95%B0%E6%8D%AE%E5%BA%93%E7%B3%BB%E7%BB%9F%E5%8E%9F%E7%90%86.md)学习笔记

