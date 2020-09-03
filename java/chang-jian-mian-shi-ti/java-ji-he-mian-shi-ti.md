# JAVA集合面试题

## **Java集合的框架有哪几种**

 两种map和collection，其中collection分为set和list.

## **Vector, ArrayList和LinkedList的区别**

* ArrayList的实现是一个可变数组，默认的初始长度为10，也可以我们自己设置容量，但是没有设置的时候默认的是空数组，只有在第一步add的时候会进行扩容至10（重新创建了数组），后面按照3/2的大小进行扩容，使线程不安全的，使用多读取、少插入的情况。
* LinkedList是基于双向链表的实现，使用了尾插法的方式，内部维护了链表的长度，以及头节点和尾节点，所以获取长度不需要遍历，适合一些插入/删除频繁的情况。
* Vector是线程安全的，实现方式和ArrayList相似，也是基于数组，但是方法上面有synchronized关键词修饰，其扩容方式是原来的两倍。

## **HashMap, HashTable和ConcurrentHashMap的区别**

* HashMap是Map类型的一种最常用的数据结构，其底部实现是数组+链表（在1.8版本后变成了数组+链表/红黑树的方法），其key可以是null，默认hash值为0，以2的幂等次进行扩容，时线程不安全的。
* HashTable的实现形式和HashMap差不多，它是线程安全的，继承了Dictionary，也是key-value的模式，但是其key不能为null。
* ConcurrentHashMap是JUC并发包中的一种，在HashMap的基础上进行了修改，因为HashMap其实是线程不安全的，而HashTable虽然是线程安全的，但是HashTable是全程加锁的，因而性能不好（是Java遗留类），ConcurrentHashMap采用分段的思想，把原本一个数组分成默认的16段，就可以最多容纳16个线程并发操作，着16个段叫做Segment，是基于ReentrantLock来实现的。

## **谈谈HashMap**

HashMap是Map结构，内部使用了数组+链表的方式，在1.8后，当链表长度达到8的时候，会变成红黑树，这样就可以把查询的复杂度变成O\(nlogn\)，默认负载因子为0.75。（负载因子为0.75的原因：当负载因子太小时，很容易触发扩容；负载因子太大则容易出现碰撞，所以这是一个空间和时间的均衡点，在1.8的HashMap介绍中提到，0.75的负载因子能让随机hash更加满足0.5的泊松分布）

此外，1.7的时候时头插法，1.8后变成了尾插法，主要是为了解决rehash出现的死循环问题，而且1.7的时候时先扩容再插入，1.8则是先插入再扩容（原因：因为读写问题，HashMap并不是线程安全的，如果先扩容再写入，那在扩容期间，是访问不到新放入的值的，因此先插入，这样在扩容期间新插入的值也是存在的）

1.7版本使用了9次扰动，5次异或，4次位移，减少hash冲突，但是1.8只使用了一次异或，一次位移。

## **谈谈ConcurrentHashMap**

concurrentHashMap是线程安全的map结构，它的核心思想是分段锁。在1.7版本的时候，内部维护了segment数组，默认是16个，segment中有一个Table（相当于一个segment存放一个HashMap），segment锁继承了ReentrantLock，使用了互斥锁，ConcurrentHashMap的size其实就是segment数组的count和。而在1.8的时候做了一个大改版，废除了segment，采用了CAS加synchronized方式进行分段锁（还有自旋锁的保证），而且节点对象改用了Node替换HashEntry。

Node可以支持链表和红黑树的转化，比如TreeBin就是继承了Node，这样子可以直接用instanceof来区分。1.8的put就很复杂，会先计算hash值，然后根据hash值选出Node数组的下标（默认数组是空，所以一开始put的时候会初始化，指定负载因子是0.75，不可变），判断是否为空，如果为空，则用cas操作来赋值首节点，如果失败，则因为自旋，会进入非空节点的逻辑，这个时候会用synchronized加锁头节点（保证整条链路锁定）这个时候还会进行二次判断，是否是同一个首节点，再分首节点是链表还是树结构，进行遍历判断。

## **ConcurrentHashMap的扩容方式**

1.7版本的ConcurrentHashMap的基于segment的，segment内部维护了HashEntry数组，所以扩容是在这个基础上的，类比HashMap的扩容。

1.8版本的ConcurrentHashMap扩容方式比较复杂，利用了ForwardingNode，会先根据机器内核数来分配每个线程能分到的busket数（最小是16）,这样可以做到多携程协助迁移，提升速递，然后更具自己分配的busket数来进行节点转移，如果为空，就放置ForwardingNode，代表已经迁移完成，如果是非空节点，加锁，链路循环，进行迁移。

## **HashMap的put方法过程**

判断key是否是null，如果是null对应的hash值是0，获得hash值过后则进行扰动，（1.7是9次扰动，5次异或，4次位移；1.8是2次扰动，一次异或，一次位移），获得的新hash值找出所在的index，\(n-1\)&hash，根据下表找到对应的Node/Entry，然后遍历链表/红黑树，如果遇到hash值相同且equals相同，则覆盖值，如果不是则新增。如果节点数大于8了，则进行数化（1.8）。完成后，判断当前的长度是否大于阈值，是就扩容（先插入再扩容是1.8的操作，而1.7是先扩容再put）。

## **为什么修改hashcode必须要修改equals**

与map有关，在map中判断是否是同一个对象时，会先判断hash值，再判断equals，如果只重写了hashcode而没有重写equals，而是使用Object的equals，那么就是判断两者指针是否一致，则会出现两个值相同对象对于map而言是两个不同对象从而出现问题。

## **谈谈TreeMap**

TreeMap是map中的一种特殊的map，我们知道map基本是无序的，但是TreeMap是会自动进行排序的，也就是一个有序的Map（使用红黑树来实现），如果设置了Comparator比较器，则会根据比较器来比较两者的大小，如果没有则key需要是Comparable的子类（代码没有实现检查，会直接抛出转化异常）

## **谈谈LinkedHashMap**

LinkedHashMap是HashMap的一种特殊分支，是某种有序的HashMap，和TreeMap不一的是，LinkedHashMap采用HashMap+双向链表的方式来构造，有两种有序模式：访问有序和插入有序。当accessOrder设置为false，表示不是访问有序而是插入有序，这也是默认值，表明LinkedHashMap中存储的顺序是按照调用put方式插入的顺序进行排序的；当accessOrder设置为true时，每次get的时候都会把链表的节点移除掉，放到链表的最后面，这样就是LRU的一种实现方式。

