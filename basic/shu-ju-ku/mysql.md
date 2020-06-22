# MySQL

## 一、索引 <a id="%E4%B8%80%E3%80%81%E7%B4%A2%E5%BC%95"></a>

### B+ Tree原理 <a id="B%2B%20Tree%E5%8E%9F%E7%90%86"></a>

**1. 数据结构**

B Tree指的是Balance Tree，也就是平衡树。平衡树是一颗查找树，并且所有的叶子节点位于同一层。

B+ Tree是基于B Tree和叶子节点顺序访问指针进行实现，它具有B Tree的平衡性，并且通过顺序访问指针来提高区间查找的性能。

在B+ Tree中，一个节点中的key从左到右非递减排列，如果某个指针的左右相邻key分别是![key\_i](https://private.codecogs.com/gif.latex?key_i)和![key\_{i+1}](https://private.codecogs.com/gif.latex?key_%7Bi&plus;1%7D)且不为null，则该指针指向节点的所有key大于等于![key\_i](https://private.codecogs.com/gif.latex?key_i)且小于等于![key\_{i+1}](https://private.codecogs.com/gif.latex?key_%7Bi&plus;1%7D)。

![](https://img-blog.csdnimg.cn/20200429094725781.png)

 **2. 操作**

进行查找操作时，首先在根节点进行二分查找，找到一个key所在的指针，然后递归地在指针所指向的节点进行查找。知道查找到叶子节点，然后在叶子节点上进行二分查找，找出key所对应的data。

插入删除操作回破坏平衡树的平衡性，因此在插入删除操作之后，需要对树进行一个分裂、合并、旋转等操作来维护平衡性。

**3. 与红黑树的比较**

红黑树等平衡树也可以用来实现索引，但是文件系统及数据库系统普遍采用B+ Tree作为索引结构，主要有以下两个原因：

（一）更少的查找次数

平衡树查找操作的时间复杂度和树高h相关，![O\(h\)=O\(log\_d N\)](https://private.codecogs.com/gif.latex?O%28h%29%3DO%28log_d%20N%29)，其中d为每个节点的出度。

红黑树的出度为2，而B+ Tree的出度一般都非常大，所以红黑树的树高h很明显比B+ Tree大非常多，查找的次数也就多非常多。

（二）利用磁盘预读特性

为了减少磁盘I/O操作，磁盘往往不是严格按需读取，而是每次都会预读。预读过程中，磁盘进行顺序读取，顺序读取不需要进行磁盘寻道，并且只需要很短的磁盘旋转时间，速度会非常块。

操作系统一般将内存和磁盘分割成固定大小的块，每一块称为一页，内存与磁盘以页为单位交换数据。数据库系统将索引的一个节点大小设置为页的大小，使得一次I/O就能完全载入一个节点，并且可以利用预读特性，相邻的节点也能够被预先载入。

### MySQL索引 <a id="MySQL%E7%B4%A2%E5%BC%95"></a>

索引是在存储引擎实现的，而不是在服务器层实现的，所以不同存储引擎具有不同的索引类型和实现。

**1. B+ Tree索引**

是大多数MySQL存储引擎的默认索引类型。

因为不再需要进行全表扫描，只需要对树进行搜索即可，所以查找速度快很多。

因为B+ Tree的有序性，所以除了用于查找，还可以用于排序和分组。

可以指定多个列作为索引列，多个索引列共同组成键。

适用于全键值、键值范围和键前缀查找，其中键前缀查找只适用于最左前缀查找。如果不是按照索引列的顺序进行查找，则无法使用索引。

InnoDB的B+ Tree索引分为主索引和辅助索引。主索引的叶子节点data域记录着完整的数据记录，这种索引方式被称为聚簇索引。

![](https://img-blog.csdnimg.cn/20200429100059996.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMwMjQxNzA5,size_16,color_FFFFFF,t_70)

辅助索引的叶子节点的data域记录着主键的值，因此在使用辅助索引进行查找时，需要先查找到主键值，然后再到主索引中进行查找。

![](https://img-blog.csdnimg.cn/20200429100615149.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMwMjQxNzA5,size_16,color_FFFFFF,t_70)

 **2. 哈希索引**

哈希索引能以O\(1\)时间进行查找，但是失去了有序性：

* 无法用于排序和分组
* 只支持精确查找，无用用于部分查找和范围查找。

InnoDB存储引擎有一个特殊的功能叫"自适应哈希索引"，当某个索引值被使用得非常频繁时，会在B+ Tree索引值上再创建一个哈希索引，这样就让B+ Tree索引具有哈希索引得一些优点，比如快速的哈希查找。

**3. 全文索引**

MyISAM存储引擎支持全文索引，用于查找文本中的关键词，而不是直接比较是否相等。

查找条件使用MATCH AGAINST，而不是普通的WHERE。

全文索引使用倒排索引实现，它记录着关键词到其所在文档的映射。

InnoDB存储引擎在MySQL 5.6.4版本中也开始支持全文索引。

**4. 空间数据索引**

MyISAM存储引擎支持空间数据索引（R-Tree），也可以用于地理数据存储。空间数据索引会从所有维度来索引数据，可以有效地使用任意维度来进行组合查询。

必须使用GIS相关的函数来维护数据。

### 索引优化 <a id="%E7%B4%A2%E5%BC%95%E4%BC%98%E5%8C%96"></a>

**1. 独立的列**

在进行查询时，索引列不能时表达式的一部分，也不能是函数的参数，否则无法使用索引。

例如下面的查询不能使用actor\_id列的索引：

```text
SELECT actor_id FROM sakila.actor WHERE actor_id + 1 = 5;
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

**2. 多列索引** 

在需要使用多个列作为条件进行查询时，使用多列索引比使用多个单列索引性能更好。

例如下面的语句中，最好把actor\_id和film\_id设置为多列索引。

```text
SELECT file_id, actor_id FROM sakila.film_actor
WHERE actor_id = 1 AND film_id = 1;
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

 **3. 索引列的顺序**

让选择性最强的索引列放在前面。

索引的选择性是指：不重复的索引值和记录值的比值。最大值为1，此时每个记录都有唯一的索引与其对应。选择性越高，每个记录的区分度越高，查询效率也越高。

例如下面显示的结果中customer\_id的选择性比staff\_id更高，因此最好把customer\_id列放在多列索引的前面。

```text
SELECT COUNT(DISTINCT staff_id) / COUNT(*) AS staff_id_selectivity,
COUNT(DISTINCT customer_id) / COUNT(*) AS costomer_id_selectivity,
COUNT(*)
FROM payment;

   staff_id_selectivity: 0.0001
customer_id_selectivity: 0.0373
               COUNT(*): 16049
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

**4. 前缀索引**

对于BLOB, TEXT和VARCHAR类型的列，必须使用前缀索引，之索引开始的部分字符。

前缀长度的选取需要根据索引选择性来确定。

**5. 覆盖索引**

索引包含所有需要查询的字段的值。

覆盖索引具有以下优点：

* 索引行通常小于数据行的大小，只读取索引能大大减少数据访问量。
* 一些存储引擎（例如MyISAM）在内存中只缓存索引，而数据依赖于操作系统来缓存。因此，只访问索引可以不适用系统调用（通常比较费时）。
* 对于InnoDB引擎，若辅助索引能够覆盖查询，则无需访问主索引。

### 索引的优点 <a id="%E7%B4%A2%E5%BC%95%E7%9A%84%E4%BC%98%E7%82%B9"></a>

* 大大较少了服务器需要扫描的数据行数。
* 帮助服务器避免进行排序和分组，以及避免创建临时表（B+ Tree索引是有序的，可以用于ORDER BY和GROUP BY操作。临时表主要是在排序和分组过程中创建，不需要排序和分组，也就不需要创建临时表）。
* 将随机I/O变为顺序I/O（B+ Tree索引是有序的，会将相邻的数据都存储在一起）。

### 索引的使用条件 <a id="%E7%B4%A2%E5%BC%95%E7%9A%84%E4%BD%BF%E7%94%A8%E6%9D%A1%E4%BB%B6"></a>

* 对于非常小的表、大部分情况下简单的全表扫描比建立索引更高效；
* 对于中到大型的表，索引就非常有效；
* 但是对于特大型的表，建立和维护索引的代价将会随之增长。这种情况下，需要用到一种技术可以直接区分出需要查询的一组数据，而不是一条记录一条记录地匹配，例如可以使用分区技术。

## 二、查询性能优化 <a id="%E4%BA%8C%E3%80%81%E6%9F%A5%E8%AF%A2%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96"></a>

### 使用Explain进行分析 <a id="%E4%BD%BF%E7%94%A8Explain%E8%BF%9B%E8%A1%8C%E5%88%86%E6%9E%90"></a>

Explain用来分析SELECT查询语句，可以通过分析Explain结果来优化查询语句。

比较重要的字段有：

* select\_type: 查询类型，有简单查询、联合查询、子查询等
* key：使用的索引
* rows：扫描的行数

### 优化数据访问 <a id="%E4%BC%98%E5%8C%96%E6%95%B0%E6%8D%AE%E8%AE%BF%E9%97%AE"></a>

**1.减少请求的数据量**

* 只返回必要的列：最好不要使用SELECT \* 语句
* 只返回必要的行：使用LIMIT语句来限制返回的数据
* 缓存重复查询的数据：使用缓存可以避免在数据库中进行查询，特别是在要查询的数据经常被重复查询时，缓存带来的查询性能提升将会是非常名相的。

**2. 减少服务器端扫描的行数**

最有效的方式是使用索引来覆盖查询。

### 重构查询方式 <a id="%E9%87%8D%E6%9E%84%E6%9F%A5%E8%AF%A2%E6%96%B9%E5%BC%8F"></a>

**1. 切分大查询**

一个大查询如果一次性执行的话，可能一次锁住很多数据、占满整个事务日志、耗尽资源系统、阻塞很多小的但是很重要的查询。

```text
DELETE FROM messages WHERE create <DATE_SUB(NOW(), INTERVAL 3 MONTH);
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

 切分为：

```text
rows_affected = 0
do {
    rows_affected = do_query(
        "DELETE FROM messages WHERE create <DATE_SUB(NOW(), INTERVAL 3 MONTH) LIMIT 10000"
)
} while rows_affected > 0
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

 **2. 分解大连接查询**

将一个大连接查询分解成对每一个表进行一次单表查询，然后在应用程序中进行关联，这样做的好处有：

* 让缓存更高效。对于连接查询，如果其中一个表发生变化，那么整个查询缓存就无法使用。而分解后的多个查询，即使其中一个表发生变化，对其它表的查询缓存依然可以使用。
* 分解成多个单表查询，这些单表查询的缓存结果更可能被其他查询使用到，从而减少冗余记录的查询。
* 减少锁竞争。
* 在应用层进行连接，可以更容易地对数据库进行拆分，从而更容易做到高性能和可伸缩。
* 查询本身效率也可能会有所提升。例如下面的例子中，使用IN\(\)代替连接查询，可以让MySQL按照ID顺序进行查询，这可能比随机的连接要更高效。

```text
SELECT * FROM tag
JOIN tag_post ON tag_post.tag_id = tag.id
JOIN post ON tag_post.post_id = post.id
WHERE tag.tag = 'mysql';
```

```text
SELECT * FROM tag WHERE tag = 'mysql';
SELECT * FROM tag_post WHERE tag_id = 1234;
SELECT * FROM post WHERE post.id IN (123, 456, 567, 9098, 8904);
```

## 三、存储引擎 <a id="%E4%B8%89%E3%80%81%E5%AD%98%E5%82%A8%E5%BC%95%E6%93%8E"></a>

### InnoDB <a id="InnoDB"></a>

InnoDB是MySQL默认的事务型存储引擎，只有在需要它不支持的特性时，才考虑使用其它存储引擎。

实现了四个标准的隔离级别，默认级别是可重复读（REPEATABLE READ）。在可重复读隔离级别下，通过多版本并发控制（MVCC）+Next-Key Locking防止幻影读。

主索引是聚簇索引，在索引中保存了数据，从而避免直接读取磁盘，因此对查询性能有较大的提升。

内部做了很多优化，包括从磁盘读取数据时采用的可预测性读、能够加快读操作并且自动创建的自适应哈希索引、能够加速插入操作的插入缓存区等。

支持真正的在线热备份。其它存储引擎不支持在线热备份，要获取一致性视图需要停止对所有表的写入，而在读写混合场景中，停止写入可能意味着停止读取。

### MyISAM <a id="MyISAM"></a>

设计简单，数据以紧密格式存储。对于只读数据，或者表比较小、可以容忍修复操作，则依然可以使用它。

提供了大量的特性，包括压缩表、空间数据索引等。

不支持事务。

不支持行级锁，只能对整张表加锁，读取时会对需要读到的所有表加共享锁，写入时则对表加排它锁。但在表有读取操作的同时，也可以往表中插入新的记录，这被称为并发插入（CONCURRENT INSERT）。

可以手动或者自动执行检查和修复操作，但是和事务恢复以及崩溃恢复不同，可能导致一些数据丢失，而且修复操作是非常慢的。

如果指定了DELAY\_KEY\_WRITE选项，在每次修改执行完成时，不会立即将修改的索引数据写入磁盘，而是会写入到内存中的缓冲区，只有在清理键缓冲区或者关闭表的时候才会将对应的索引块写入磁盘。这种方式可以极大的提升写入性能，但是在数据库或者主机崩溃时会造成索引损坏，需要执行修复操作。

### 比较 <a id="%E6%AF%94%E8%BE%83"></a>

* 事务：InnoDB是事务型的，可以使用Commit和Rollback语句。
* 并发：MyISAM只支持表级锁，而InnoDB还支持行级锁。
* 外键：InnoDB支持外键。
* 备份：InnoDB支持在线热备份。
* 崩溃恢复：MyISAM崩溃后发生损坏的概率比InnoDB高很多，而且恢复的速度也更慢。
* 其它特性：MyISAM支持压缩表和空间数据索引。

## 四、数据类型 <a id="%E5%9B%9B%E3%80%81%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B"></a>

### 整型 <a id="%E6%95%B4%E5%9E%8B"></a>

TINYINT, SMALLINT, MEDIUMINT, INT, BIGINT分别使用8, 16, 24, 32, 64位存储空间，一般情况下越小的列越好。

INT\(11\)中的数字只是规定了交互工具显示字符的个数，对于存储和计算来说是没有意义的。

### 浮点数 <a id="%E6%B5%AE%E7%82%B9%E6%95%B0"></a>

FLOAT和DOUBLE为浮点类型，DECIMAL为高精度小数类型。CPU原生支持浮点运算，但是不支持DECIMAL类型的计算，因此DECIMAL的计算比浮点类型需要更高的代价。

FLOAT, DOUBLE和DECIMAL都可以指定列宽，例如DECIMAL\(18, 9\)表示总共18位，取9位存储小数部分，剩下9位存储整数部分。

### 字符串 <a id="%E5%AD%97%E7%AC%A6%E4%B8%B2"></a>

主要有CHAR和VARCHAR两种类型，一种是定长的，一种是变长的。

VARCHAR这种变长类型能够节省空间，因为只需要存储必要的内容。但是在执行UPDATE时可能会使行变得比原来长，当超出一个页所能容纳的大小时，就要执行额外的操作。MyISAM会将行拆成不同的片段存储，而InnoDB则需要分裂页来使行放进页内。

在进行存储和检索时，会保留VARCHAR末尾的空格，而会删除CHAR末尾的空格。

### 时间和日期 <a id="%E6%97%B6%E9%97%B4%E5%92%8C%E6%97%A5%E6%9C%9F"></a>

MySQL提供了两种相似的日期时间类型：DATETIME和TIMESTAMP。

**1. DATETIME**

能够保存从1000年到9999年的日期和时间，精度为秒，使用8字节的存储空间。

它与时区无关。

默认情况下，MySQL以一种可排序的、无歧义的格式显示DATETIME值，例如“2020-04-29 11:36:04”，这是ANSI标准定义的日期和时间表示方法。

**2. TIMESTAMP**

和UNIX时间戳相同，保存从1970年1月1日午夜（格林威治时间）以来的秒数，使用4个字节，也能表示从1970年到2038年。

它和时区有关，也就是说一个时间戳在不同时区所代表的具体时间是不同的。

MySQL提供了FROM\_UNIXTIME\(\)函数把UNIX时间戳转换为日期，并提供了UNIX\_TIMESTAMP\(\)函数把日期转换为UNIX时间戳。

默认情况下，如果插入时没有指定TIMESTAMP列的值，会将这个值设置为当前时间。

应该尽量使用TIMESTAMP，因为它比DATETIME空间效率更高。

## 五、切分 <a id="%E4%BA%94%E3%80%81%E5%88%87%E5%88%86"></a>

### 水平切分 <a id="%E6%B0%B4%E5%B9%B3%E5%88%87%E5%88%86"></a>

水平切分又称为Sharding，它是将同一个表中的记录拆分到多个结构相同的表中。

当一个表的数据不断增多时，Sharding是必然的选择，它可以将数据分布到集群的不同节点上，从而缓解单个数据库的压力。

### 垂直切分 <a id="%E5%9E%82%E7%9B%B4%E5%88%87%E5%88%86"></a>

垂直切分是将一张表切分成多个表，通常是按照列的关系密集程度进行切分，也可以利用垂直切分将经常被使用的列和不经常被使用的列切分成到不同的表中。

在数据库的层面使用垂直切分将数据库表的密集程度部署到不同的库中，例如将原来的电商数据库垂直切分成商品数据库、用户数据库等。

### Sharding策略 <a id="Sharding%E7%AD%96%E7%95%A5"></a>

* 哈希取模：hash\(key\)%N
* 范围：可以是ID范围也可以是时间范围
* 映射表：使用单独的一个数据库来存储映射关系

### Sharding存在的问题 <a id="Sharding%E5%AD%98%E5%9C%A8%E7%9A%84%E9%97%AE%E9%A2%98"></a>

**1. 事务问题**

使用分布式事务来解决，比如XA接口。

**2. 连接**

可以将原来的连接分解成多个单表查询，然后在用户程序中进行连接。

**3. ID唯一性**

* 使用全局唯一ID\(GUID\)
* 为每个分片指定一个ID范围
* 分布式ID生成器（如Twitter的Snowflake算法）

## 六、复制 <a id="%E5%85%AD%E3%80%81%E5%A4%8D%E5%88%B6"></a>

### 主从复制 <a id="%E4%B8%BB%E4%BB%8E%E5%A4%8D%E5%88%B6"></a>

主要涉及三个线程：binlog线程、I/O线程和SQL线程

* binlog线程：负责将主服务器上的数据更改写入二进制日志（Binary log）中
* I/O线程：负责从主服务器上读取二进制日志，并写入从服务器的中继日志（Relay log）
* SQL线程：负责读取中继日志，解析出主服务器已经执行的数据更改并在从服务器中重放（Replay）

![](../../.gitbook/assets/image%20%2842%29.png)

### 读写分离 <a id="%E8%AF%BB%E5%86%99%E5%88%86%E7%A6%BB"></a>

主服务器处理写操作以及实时性要求比较高的读操作，而从服务器处理读操作。

读写分离能提高性能的主要原因在于：

* 主从服务器负责各自的读和写，极大程度上缓解了锁的争用；
* 从服务器可以使用MyISAM，提升查询性能以及节约系统开销；
* 增加冗余，提高可可靠性。

读写分离常用代理服务器来实现，代理服务器接收应用层传来的读写请求，然后决定转发给哪个服务器。

![](../../.gitbook/assets/image%20%2869%29.png)

