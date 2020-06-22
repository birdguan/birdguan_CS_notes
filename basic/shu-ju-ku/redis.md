# Redis

## 一、概述 <a id="%E4%B8%80%E3%80%81%E6%A6%82%E8%BF%B0"></a>

Redis是速度非常快的非关系型（NoSQL）内存键值数据库，可以存储键和五种不同类型的值之间的映射。

键的类型只能为字符串，值支持五种数据类型：字符串、列表、集合、散列表、有序集合。

Redis支持很多特性，例如将内存中的数据持久化到硬盘中，使用复制来扩展读性能，使用分片来扩展写性能。

## 二、数据类型 <a id="%E4%BA%8C%E3%80%81%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B"></a>

<table>
  <thead>
    <tr>
      <th style="text-align:left"><b>&#x6570;&#x636E;&#x7C7B;&#x578B;</b>
      </th>
      <th style="text-align:left"><b>&#x53EF;&#x4EE5;&#x5B58;&#x50A8;&#x7684;&#x503C;</b>
      </th>
      <th style="text-align:left"><b>&#x64CD;&#x4F5C;</b>
      </th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">STRING</td>
      <td style="text-align:left">&#x5B57;&#x7B26;&#x4E32;&#x3001;&#x6574;&#x6570;&#x6216;&#x6D6E;&#x70B9;&#x6570;</td>
      <td
      style="text-align:left">
        <p>&#x5BF9;&#x6574;&#x4E2A;&#x5B57;&#x7B26;&#x4E32;&#x6216;&#x5B57;&#x7B26;&#x4E32;&#x7684;&#x5176;&#x4E2D;&#x4E00;&#x90E8;&#x5206;&#x6267;&#x884C;&#x64CD;&#x4F5C;</p>
        <p>&#x5BF9;&#x6574;&#x6570;&#x548C;&#x6D6E;&#x70B9;&#x6570;&#x6267;&#x884C;&#x81EA;&#x589E;&#x6216;&#x8005;&#x81EA;&#x51CF;&#x64CD;&#x4F5C;</p>
        </td>
    </tr>
    <tr>
      <td style="text-align:left">LIST</td>
      <td style="text-align:left">&#x5217;&#x8868;</td>
      <td style="text-align:left">
        <p>&#x4ECE;&#x4E24;&#x7AEF;&#x538B;&#x5165;&#x6216;&#x8005;&#x5F39;&#x51FA;&#x5143;&#x7D20;</p>
        <p>&#x5BF9;&#x5355;&#x4E2A;&#x6216;&#x8005;&#x591A;&#x4E2A;&#x5143;&#x7D20;&#x8FDB;&#x884C;&#x4FEE;&#x526A;&#xFF0C;&#x53EA;&#x4FDD;&#x7559;&#x4E00;&#x4E2A;&#x8303;&#x56F4;&#x5185;&#x7684;&#x5143;&#x7D20;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">SET</td>
      <td style="text-align:left">&#x65E0;&#x5E8F;&#x96C6;&#x5408;</td>
      <td style="text-align:left">
        <p>&#x6DFB;&#x52A0;&#x3001;&#x83B7;&#x53D6;&#x3001;&#x79FB;&#x9664;&#x5355;&#x4E2A;&#x5143;&#x7D20;</p>
        <p>&#x68C0;&#x67E5;&#x4E00;&#x4E2A;&#x5143;&#x7D20;&#x662F;&#x5426;&#x5728;&#x96C6;&#x5408;&#x4E2D;</p>
        <p>&#x8BA1;&#x7B97;&#x4EA4;&#x96C6;&#x3001;&#x5E76;&#x96C6;&#x3001;&#x5DEE;&#x96C6;</p>
        <p>&#x4ECE;&#x96C6;&#x5408;&#x91CC;&#x9762;&#x968F;&#x673A;&#x83B7;&#x53D6;&#x5143;&#x7D20;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">HASH</td>
      <td style="text-align:left">&#x5305;&#x542B;&#x952E;&#x503C;&#x5BF9;&#x7684;&#x65E0;&#x5E8F;&#x6563;&#x5217;&#x8868;</td>
      <td
      style="text-align:left">
        <p>&#x6DFB;&#x52A0;&#x3001;&#x83B7;&#x53D6;&#x3001;&#x79FB;&#x9664;&#x5355;&#x4E2A;&#x952E;&#x503C;&#x5BF9;</p>
        <p>&#x83B7;&#x53D6;&#x6240;&#x6709;&#x952E;&#x503C;&#x5BF9;</p>
        <p>&#x68C0;&#x67E5;&#x67D0;&#x4E2A;&#x952E;&#x662F;&#x5426;&#x5B58;&#x5728;</p>
        </td>
    </tr>
    <tr>
      <td style="text-align:left">ZSET</td>
      <td style="text-align:left">&#x6709;&#x5E8F;&#x96C6;&#x5408;</td>
      <td style="text-align:left">
        <p>&#x6DFB;&#x52A0;&#x3001;&#x83B7;&#x53D6;&#x3001;&#x5220;&#x9664;&#x5143;&#x7D20;</p>
        <p>&#x6839;&#x636E;&#x5206;&#x503C;&#x8303;&#x56F4;&#x6216;&#x8005;&#x6210;&#x5458;&#x6765;&#x83B7;&#x53D6;&#x5143;&#x7D20;</p>
        <p>&#x8BA1;&#x7B97;&#x4E00;&#x4E2A;&#x952E;&#x7684;&#x6392;&#x540D;</p>
      </td>
    </tr>
  </tbody>
</table>

### STRING <a id="STRING"></a>

![](../../.gitbook/assets/image%20%2888%29.png)

```text
> set hello world
OK
> get hello
"world"
> del hello
(integer) 1
> get hello
(nil)
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

### LIST <a id="LIST"></a>

![](../../.gitbook/assets/image%20%2830%29.png)

```text
> rpush list-key item
(integer) 1
> rpush list-key item2
(integer) 2
> rpush list-key item
(integer) 3

> lrange list-key 0 -1
1) "item"
2) "item2"
3) "item"

> lindex list-key 1
"item2"

> lpop list-key
"item"

> lrange list-key 0 -1
1) "item2"
2) "item"
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

### SET <a id="SET"></a>

![](../../.gitbook/assets/image%20%2875%29.png)

```text
> sadd set-key item
(integer) 1
> sadd set-key item2
(integer) 1
> sadd set-key item3
(integer) 1
> sadd set-key item
(integer) 0

> smembers set-key
1) "item"
2) "item2"
3) "item3"

> sismember set-key item4
(integer) 0
> sismember set-key item
(integer) 1

> srem set-key item2
(integer) 1
> srem set-key item2
(integer) 0

> smembers set-key
1) "item"
2) "item3"
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

### HASH <a id="HASH"></a>

![](../../.gitbook/assets/image%20%2861%29.png)

```text
> hset hash-key sub-key1 value1
(integer) 1
> hset hash-key sub-key2 value2
(integer) 1
> hset hash-key sub-key1 value1
(integer) 0

> hgetall hash-key
1) "sub-key1"
2) "value1"
3) "sub-key2"
4) "value2"

> hdel hash-key sub-key2
(integer) 1
> hdel hash-key sub-key2
(integer) 0

> hget hash-key sub-key1
"value1"

> hgetall hash-key
1) "sub-key1"
2) "value1"
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

### ZSET <a id="ZSET"></a>

![](../../.gitbook/assets/image%20%2892%29.png)

```text
> zadd zset-key 728 member1
(integer) 1
> zadd zset-key 982 member0
(integer) 1
> zadd zset-key 982 member0
(integer) 0

> zrange zset-key 0 -1 withscores
1) "member1"
2) "728"
3) "member0"
4) "982"

> zrangebyscore zset-key 0 800 withscores
1) "member1"
2) "728"

> zrem zset-key member1
(integer) 1
> zrem zset-key member1
(integer) 0

> zrange zset-key 0 -1 withscores
1) "member0"
2) "982"
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

## 三、数据结构 <a id="%E4%B8%89%E3%80%81%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84"></a>

### 字典 <a id="%E5%AD%97%E5%85%B8"></a>

dictht是一个散列表结构，使用拉链发解决哈希冲突。

```text
/* This is our hash table structure. Every dictionary has two of this as we
 * implement incremental rehashing, for the old to the new table. */
typedef struct dictht {
    dictEntry **table;
    unsigned long size;
    unsigned long sizemask;
    unsigned long used;
} dictht;
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

```text
typedef struct dictEntry {
    void *key;
    union {
        void *val;
        uint64_t u64;
        int64_t s64;
        double d;
    } v;
    struct dictEntry *next;
} dictEntry;
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

 Redis的字典dict中包含两个哈希表dictht，这是为了方便进行rehash操作。在扩容时，将其中一个dictht上的键值对rehash到另一个hashht上面，完成之后释放空间并交换两个dictht的角色。

```text
typedef struct dict {
    dictType *type;
    void *privdata;
    dictht ht[2];
    long rehashidx; /* rehashing not in progress if rehashidx == -1 */
    unsigned long iterators; /* number of iterators currently running */
} dict;
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

rehash操作不是一次性完成，而是采用渐进式方式，这是为了避免一次性执行过多的rehash操作给服务器带来过大的负担。

渐进式rehash通过记录dict的rehashidx完成，它从0开始，然后每执行一次rehash都会递增。例如在一次rehash中，要把dict\[0\] rehash到dict\[1\]，这一次会把dict\[0\]上table\[rehashidx\]的键值对rehash到dict\[1\]上，dict\[0\]的table\[rehashidx\]指向null，并令rehashidx++。

在rehash期间，每次对字典进行添加、删除、查找或者更新操作时，都会执行一次渐进式rehash。

采用渐进式rehash会导致字典中的数据分散在两个dictht上，因此对字典的查找操作也要到对应的dictht去执行。 

```text
/* Performs N steps of incremental rehashing. Returns 1 if there are still
 * keys to move from the old to the new hash table, otherwise 0 is returned.
 *
 * Note that a rehashing step consists in moving a bucket (that may have more
 * than one key as we use chaining) from the old to the new hash table, however
 * since part of the hash table may be composed of empty spaces, it is not
 * guaranteed that this function will rehash even a single bucket, since it
 * will visit at max N*10 empty buckets in total, otherwise the amount of
 * work it does would be unbound and the function may block for a long time. */
int dictRehash(dict *d, int n) {
    int empty_visits = n * 10; /* Max number of empty buckets to visit. */
    if (!dictIsRehashing(d)) return 0;

    while (n-- && d->ht[0].used != 0) {
        dictEntry *de, *nextde;

        /* Note that rehashidx can't overflow as we are sure there are more
         * elements because ht[0].used != 0 */
        assert(d->ht[0].size > (unsigned long) d->rehashidx);
        while (d->ht[0].table[d->rehashidx] == NULL) {
            d->rehashidx++;
            if (--empty_visits == 0) return 1;
        }
        de = d->ht[0].table[d->rehashidx];
        /* Move all the keys in this bucket from the old to the new hash HT */
        while (de) {
            uint64_t h;

            nextde = de->next;
            /* Get the index in the new hash table */
            h = dictHashKey(d, de->key) & d->ht[1].sizemask;
            de->next = d->ht[1].table[h];
            d->ht[1].table[h] = de;
            d->ht[0].used--;
            d->ht[1].used++;
            de = nextde;
        }
        d->ht[0].table[d->rehashidx] = NULL;
        d->rehashidx++;
    }

    /* Check if we already rehashed the whole table... */
    if (d->ht[0].used == 0) {
        zfree(d->ht[0].table);
        d->ht[0] = d->ht[1];
        _dictReset(&d->ht[1]);
        d->rehashidx = -1;
        return 0;
    }

    /* More to rehash... */
    return 1;
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

### 跳跃表 <a id="%E8%B7%B3%E8%B7%83%E8%A1%A8"></a>

跳跃表是有序集合的底层实现之一。

跳跃表是基于多指针有序链表实现的，可以看成多个有序链表。

![](../../.gitbook/assets/image%20%2857%29.png)

在查找时，从上层指针开始查找，找到对应的区间之后再到下一层去查找。下图演示了查找22的过程。

![](../../.gitbook/assets/image%20%2811%29.png)

 与红黑树等平衡二叉树相比，跳跃表具有以下优点：

* 插入速度非常快，因为不需要进行旋转等操作来维护平衡性；
* 更容易实现；
* 支持无锁操作。

## 四、使用场景 <a id="%E5%9B%9B%E3%80%81%E4%BD%BF%E7%94%A8%E5%9C%BA%E6%99%AF"></a>

### 计数器 <a id="%E8%AE%A1%E6%95%B0%E5%99%A8"></a>

可以对String进行自增自减运算，从而实现计数器功能。

Redis这种内存型数据库的读写性能非常高，很适合存储频繁读写的计数量。

### 缓存 <a id="%E7%BC%93%E5%AD%98"></a>

将热点数据放在内存中，设置内存的最大使用量以及淘汰策略来保证缓存的命中率。

### 查找表 <a id="%E6%9F%A5%E6%89%BE%E8%A1%A8"></a>

例如DNS记录就很适合使用Redis进行存储。

查找表和缓存类似，也是利用了Redis快速的查找特性。但是查找表的内容不能失效，而缓存的内容可以失效，因为缓存不作为可靠的数据来源。

### 消息队列 <a id="%E6%B6%88%E6%81%AF%E9%98%9F%E5%88%97"></a>

List是一个双向链表，可以通过lpush和rpop写入和读取消息。

不过最好使用Kafka、RabbitMQ等消息中间件。

### 会话缓存 <a id="%E4%BC%9A%E8%AF%9D%E7%BC%93%E5%AD%98"></a>

可以使用Redis来统一存储多台应用服务器的会话信息。

当应用服务器不再存储用户的会话信息，也就不再具有状态，一个用户可以请求任意一个应用服务器，从而更容易实现高可用性以及可伸缩性。

### 分布式锁实现 <a id="%E5%88%86%E5%B8%83%E5%BC%8F%E9%94%81%E5%AE%9E%E7%8E%B0"></a>

在分布式场景下，无法使用单机环境下的锁来对多个节点上的进程进行同步。

可以使用Redis自带的SETNX命令实现分布式锁，除此之外，还可以使用官方提供的RedLock分布式所实现。

### 其它 <a id="%E5%85%B6%E5%AE%83"></a>

Set可以实现交基、并集等操作，从而实现共同好友等功能。

ZSet可以实现有序性操作，从而实现排行榜等功能。

## 五、Redis与Memcached <a id="%E4%BA%94%E3%80%81Redis%E4%B8%8EMemcached"></a>

两者都是非关系型内存键值数据库，主要有以下不同：

### 数据类型 <a id="%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B"></a>

Memcached仅支持字符串类型，而Redis支持五种不同的数据类型，可以更灵活地解决问题。

### 数据持久化 <a id="%E6%95%B0%E6%8D%AE%E6%8C%81%E4%B9%85%E5%8C%96"></a>

Redis支持两种持久化策略：RDB快照和AOF日志，而Memcached不支持持久化。

### 分布式 <a id="%E5%88%86%E5%B8%83%E5%BC%8F"></a>

Memcached不支持分布式，只能通过在客户端使用一致性哈希来实现分布式存储，这种方式在存储和查询时都需要现在客户端计算一次数据所在的节点。

Redis Cluster实现了分布式的支持。

### 内存管理机制 <a id="%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86%E6%9C%BA%E5%88%B6"></a>

在Redis中，并不是所有数据都一直存储在内存中，可以将一些很久没有使用的value交换到磁盘，而Memcached得数据则一直在内存中。

Memcached将内存分割成特定长度的块来存储数据，以完全解决内存碎片的问题。但是这种方式会使得内存的利用率不高，例如块的大小为128bytes，只存储100bytes的数据，那么剩下的28bytes就浪费掉了。

## 六、键的过期时间 <a id="%E5%85%AD%E3%80%81%E9%94%AE%E7%9A%84%E8%BF%87%E6%9C%9F%E6%97%B6%E9%97%B4"></a>

Redis可以为每个键设置过期事件，当键过期时，会自动删除该键。

对于散列表这种容器，只能为整个键设置过期时间（整个散列表），而不能为键里面的单个元素设置过期时间。

## 七、数据淘汰策略 <a id="%E4%B8%83%E3%80%81%E6%95%B0%E6%8D%AE%E6%B7%98%E6%B1%B0%E7%AD%96%E7%95%A5"></a>

可以设置内存最大使用量，当内存使用量超出时，会施行数据淘汰策略。

Redis具体有6中淘汰策略：

| **策略** | **描述** |
| :--- | :--- |
| volatile-lru | 从已设置过期时间的数据集中挑选最近最少使用的数据淘汰 |
| volatile-ttl | 从已设置过期时间的数据集中挑选将要过期的数据淘汰 |
| volatile-random | 从以设置过期时间的数据集中任意选择数据淘汰 |
| allkeys-lru | 从所有数据集中挑选最近最少使用的数据集淘汰 |
| allkeys-random | 从所有数据集中任意选择数据进行淘汰 |
| noeviction | 精致驱逐数据 |

作为内存数据库，出于对性能和内存消耗的考虑，Redis的淘汰算法实际上并非针对所有key，而是抽样一小部分并且从中选出被淘汰的key。

使用Redis缓存数据时，为了提高缓存命中率，需要保证缓存数据都是热点数据。可以将内存最大使用量设置为热点数据占用的内存量，然后启动allkeys-lru淘汰策略，将最近最少使用的数据淘汰。

 Redis 4.0引入了volatile-lfu和allkeys-lfu淘汰策略，LFU策略通过统计访问频率，将访问频率最少的键值对淘汰。

## 八、持久化 <a id="%E5%85%AB%E3%80%81%E6%8C%81%E4%B9%85%E5%8C%96"></a>

Redis是内存型数据库，为了保证数据在断电后不会丢失，需要将内存中的数据持久化到硬盘上。

### RDB持久化 <a id="RDB%E6%8C%81%E4%B9%85%E5%8C%96"></a>

将某个时间点的所有数据都存放到硬盘上。

可以将快照复制到其它服务器从而创建具有相同数据的服务器副本。

如果系统发生故障，将会丢失最后一次创建快照之后的数据。

如果数据量很大，保存快照的时间会很长。

### AOF持久化 <a id="AOF%E6%8C%81%E4%B9%85%E5%8C%96"></a>

将写命令添加到AOF文件（Append Only File）的末尾。

使用AOF持久化需要设置同步选项，从而确保写命令同步到磁盘文件上的时机。这是因为对文件进行写入并不会马上将内容同步到磁盘上，而是先存储到缓冲区，然后由操作系统决定什么时候同步到磁盘。有以下同步选项：

| 选项 | 同步频率 |
| :--- | :--- |
| always | 每个写命令都同步 |
| everysec | 每秒同步一次 |
| no | 让操作系统来决定何时同步 |

*  always选项会严重降低服务器的性能
* everysec选项比较适合，可以保证系统崩溃时只会丢失一秒左右的数据，并且Redis每秒只从一次同步对服务器性能几乎没有任何影响；
* no选项并不能给服务器性能带来多大的提升，而且也会增加系统崩溃时数据丢失的数量。

随着服务器读写请求的增多，AOF文件会越来越大。Redis提供了一种将AOF重写的特性，能够去除AOF文件中的冗余写命令。

## 九、事务 <a id="%E4%B9%9D%E3%80%81%E4%BA%8B%E5%8A%A1"></a>

一个事务包含了多个命令，服务器在执行事务期间，不会该去执行其它客户端的命令请求。

事务中的多个命令被一次性发送给服务器，而不是一条一条发送，这种方式被称为流水线，它可以减少客户端与服务器之间的网络通信次数从而提升性能。

Redis最简单的事务实现方式是使用MULTI和EXEC命令将事务操作包围起来。

## 十、事件 <a id="%E5%8D%81%E3%80%81%E4%BA%8B%E4%BB%B6"></a>

Redis服务器是一个事件驱动程序。

### 文件事件 <a id="%E6%96%87%E4%BB%B6%E4%BA%8B%E4%BB%B6"></a>

服务器通过套接字与客户端或者其它服务器进行通信，文件事件就是对套接字操作的抽象。Redis基于Reactor模式开发了自己的网络事件处理器，使用I/O多路复用程序来同时监听多个套接字，并将到达的事件传送给文件文件分派器，分派器会根据套接字产生的事件类型条用响应的事件处理器。

![](../../.gitbook/assets/image%20%2883%29.png)

### 时间事件 <a id="%E6%97%B6%E9%97%B4%E4%BA%8B%E4%BB%B6"></a>

服务器有一些操作需要在给定的时间点执行，时间事件是对这类定时操作的抽象。时间事件又分为：

* 定时事件：是一段让程序在指定时间之内执行一次；
* 周期性事件：是让一段程序每隔指定事件就执行一次。

Redis将所有时间事件都放在一个无序链表中，通过遍历整个链表查找出已到达的时间事件，并调用相应的事件处理器。

### 事件的调度与执行 <a id="%E4%BA%8B%E4%BB%B6%E7%9A%84%E8%B0%83%E5%BA%A6%E4%B8%8E%E6%89%A7%E8%A1%8C"></a>

服务器需要不断监听文件事件的套接字才能得到待处理的文件事件，但是不能一直监听，否则时间事件无法在规定的时间内执行，因此监听时间应该根据距离现在最近的的时间事件来决定。

事件调度与执行由aeProcessEvents函数负责，伪代码如下：

```text
def aeProcessEvents():
    # 获取到达时间离当前时间最接近的时间事件
    time_event = aeSearchNearestTimer()
    # 计算最接近的时间事件距离到达还有多少毫秒
    remaind_ms = time_event.when - unix_ts_now()
    # 如果事件已到达，那么 remaind_ms 的值可能为负数，将它设为 0
    if remaind_ms < 0:
        remaind_ms = 0
    # 根据 remaind_ms 的值，创建 timeval
    timeval = create_timeval_with_ms(remaind_ms)
    # 阻塞并等待文件事件产生，最大阻塞时间由传入的 timeval 决定
    aeApiPoll(timeval)
    # 处理所有已产生的文件事件
    procesFileEvents()
    # 处理所有已到达的时间事件
    processTimeEvents()
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

将asProcessEvents函数置于一个循环里面，加上初始化和清理函数，就构成了Redis服务器的主函数，伪代码如下：

```text
def main():
    # 初始化服务器
    init_server()
    # 一直处理事件，直到服务器关闭为止
    while server_is_not_shutdown():
        aeProcessEvents()
    # 服务器关闭，执行清理操作
    clean_server()
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

 从事件处理的角度来看，服务器运行流程如下：

![](../../.gitbook/assets/image%20%285%29.png)

## 十一、复制 <a id="%E5%8D%81%E4%B8%80%E3%80%81%E5%A4%8D%E5%88%B6"></a>

通过使用slaveof host port命令来让一个服务器称为另一个服务器的从服务器。

一个从服务器只能有一个主服务器，并且不支持主主复制。

### 连接过程 <a id="%E8%BF%9E%E6%8E%A5%E8%BF%87%E7%A8%8B"></a>

1. 主服务器创建快照文件，发送给从服务器，并在发送期间使用缓冲区记录执行的写命令。快照文件发送完毕之后，开始向从服务器发送存储在缓冲区中的写命令；
2. 从服务器丢弃所有的旧数据，载入主服务器发来的快照文件，之后从服务器开始接受主服务器发来的写命令；
3. 主服务器没执行一次写命令，就像从服务器发送相同的写命令。

### 主从链 <a id="%E4%B8%BB%E4%BB%8E%E9%93%BE"></a>

随着负载不断上升，主服务器可能无法很快地更新所有的从服务器，或者重新连接和重新同步从服务器将导致系统超载。为了解决这个问题，可以创建一个中间层来分担主服务器的复制工作。中间层的服务器是最上层服务器的从服务器，又是最下层服务器的主服务器。

![](../../.gitbook/assets/image%20%2815%29.png)

## 十二、Sentinel <a id="%E5%8D%81%E4%BA%8C%E3%80%81Sentinel"></a>

Sentinel（哨兵）可以监听集群中的服务器，并在主服务器进入下线状态时，自动从从服务器中选举出新的主服务器。

## 十三、分片 <a id="%E5%8D%81%E4%B8%89%E3%80%81%E5%88%86%E7%89%87"></a>

分片是将数据划分为多个部分的方法，可以将数据存储到多台机器里面，这种方法在解决某些问题时可以获得线性级别的性能提升。

假设有四个Redis实例R0, R1, R2, R3，还有很多表示用户的键user:1, user:2, ... 有不同的方式来选择一个指定的键存储在哪个实例中。

* 最简单的方式是范围分片，例如用户id从0~1000的存储到实例R0中，用户id从1001到2000的存储到实例R1中等。但是这样需要维护一张映射范围表，维护操作代价很高。
* 还以一种方式是哈希分片，使用CRC32哈希函数将键转换为一个数字，再对实例数量求模就能知道应该存储的实例。

根据执行分片的位置，可以分为三种分片方式：

* 客户端分片：客户端使用一致性哈希等算法决定键应当分布到哪个节点。
* 代理分片：将客户端请求发送到代理上，由代理转发请求到正确的节点上。
* 服务器分片：Redis Cluster

## 十四、一个简单的论坛系统分析 <a id="%E5%8D%81%E5%9B%9B%E3%80%81%E4%B8%80%E4%B8%AA%E7%AE%80%E5%8D%95%E7%9A%84%E8%AE%BA%E5%9D%9B%E7%B3%BB%E7%BB%9F%E5%88%86%E6%9E%90"></a>

该论坛系统功能如下：

* 可以发布文章；
* 可以对文章进行点赞；
* 在首页可以按文章的发布时间或者文章的点赞数进行排序显示。

### 文章信息 <a id="%E6%96%87%E7%AB%A0%E4%BF%A1%E6%81%AF"></a>

文章包括标题、作者、赞数等信息，在关系型数据库中很容易构建一张表来存储这些信息，在Redis中可以使用HASH来存储每种信息以及其对应的值的映射。

Redis没有关系型数据库中的表这一概念来将同种类型的数据存放在一起，而是使用命名空间的方式来实现这一功能。键名的前面部分存储命名空间，后面部分的内存存储ID，通常使用: 来进行分隔。例如下面的HASH的键名为article:982617，其中article为命名空间，ID为982617。

![](../../.gitbook/assets/image%20%2813%29.png)

### 点赞功能 <a id="%E7%82%B9%E8%B5%9E%E5%8A%9F%E8%83%BD"></a>

当有用户为一篇文章点赞时，除了要对该文章的votes字段进行加1操作，还必须记录该用户已经对文章进行了点赞，防止用户点赞次数超过1。可以建立文章的已投票用户集合来进行记录。

为了节约内存，规定一篇文章发布满一周之后，就不能再对它进行投票，而文章的已投票集合也会被删除，可以为文章的已投票集合设置一个一周的过期时间就能实现这个功能。

![](../../.gitbook/assets/image%20%2887%29.png)

### 对文章进行排序 <a id="%E5%AF%B9%E6%96%87%E7%AB%A0%E8%BF%9B%E8%A1%8C%E6%8E%92%E5%BA%8F"></a>

为了按发布时间和点赞数进行排序，可以建立一个文章发布时间的有序集合和一个文章点赞数的有序集合。

下图中的score就是点赞数，有序集合中的分值并不直接是时间和点赞数，而是根据时间和点赞数间接计算出来的）

![](../../.gitbook/assets/image%20%28110%29.png)

