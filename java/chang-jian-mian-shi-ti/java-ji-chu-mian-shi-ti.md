# JAVA基础面试题

## AVL树与红黑树（R-B树）的区别和联系

* AVL树是严格的平衡树 ，因此在增加或者删除节点的时候，根据不同情况，旋转次数比红黑树要多；
* 红黑树是非严格的平衡来换取增删节点时候旋转次数的开销降低；
* 所以总结说，查询次数多选择AVL树，查询更新次数差不多选择红黑树；
* AVL树顺序插入和删除时有20%左右的性能优势，红黑树随机操作时有15%左右的优势，现实应用当然一般都是随机情况，所以红黑树得到了更广泛的应用，索引为B+树，Hashmap为红黑树。

## 为啥redis zset使用跳跃表而不用红黑树实现

* skiplist的复杂度和红黑树一样，而且实现起来更简单
* 在并发环境下红黑树在插入和删除时需要rebalance，性能不如跳跃表

## JAVA基本数据类型

* 整数型：byte\(1字节\)、short\(2字节\)、int\(4字节\)、long\(8字节\)
* 浮点型：float\(4字节\)、double\(8字节\)
* 布尔型：boolean\(1字节\)
* 字符型：char\(2字节\)

## IO和NIO

* IO包括File, OutputStream, InputStream, Writer, Reader, Seralizable（5类1接口）
* NIO三大核心内容selector（选择器，用于监听channel），channel（通道） ，buffer（缓冲区）
* NIO与IO的区别：
  * IO面向流，NIO面向缓冲区
  * IO阻塞，NIO非阻塞

## **异常类**

Throwable为父类，子类有Error和Exception

Exception分RuntimeException（空指针、越界等）和CheckException（sql, io, 找不到类等）

## **LVS（4层与7层）原理**

* 由前端虚拟负载均衡器和后端真实服务器群组成
* 请求发送给虚拟服务器后根据包转发策略以及负载均衡调赴算法发送给真实服务器
* 所谓四层（LVS, f5）就是基于IP+端口的负载均衡，七层（nginx）就是基于URL等应用层信息的负载均衡

## **StringBuilder与StringBuffer**

* StringBuilder更快
* StringBuffer是线程安全的

## **interrupt/isInterrupted/interrupted区别**

* interrupt\(\): 调用该方法的线程状态被置为“中断”状态
* isInterrupted\(\): 作用于调用该方法的线程对象对应的线程的中断信号是true还是false
* interrupted\(\): 静态方法，内部实现是调用当前线程的isInterrupted\(\)，并且会重置当前线程的中断状态。

## **sleep与wait区别**

sleep属于线程类，wait属于Object类，sleep不释放锁。

## **CountDownLatch和CyclicBarrier区别**

* CountDownLatch用于主线程等待其他子线程任务都执行完毕后再执行，CyclicBarrier用于一组线程相互等待大家都达到某个状态后，再同时执行；
* CountDownLatch是不可重用的，CyclicBarrier是可重用的

## **终止线程方法**

* 使用退出标志
* 通过判断this.interrupted\(\) throw new InterruptedException\(\)来停止

## **ThreadLocal的原理和应用**

⭐ 原理：

线程中创建副本，访问自己内部的副本变量，内部实现是其内部类名叫ThreadLocalMap的成员变量threadLocals，key为本身，value为实际存值的变量副本

⭐ 应用：

* 用来解决数据库连接，存放connection对象，不同线程存放各自session
* 解决simpleDateFormat线程安全问题
* 会出现内存泄漏，显式remove不要与线程池配合，因为worker往往不会退出

## **ThreadLocal内存泄漏问题**

如果是强引用，设置tl=null，但是key的引用依然指向ThreadLocal对象，所以会有内存泄漏，而使用弱引用则不会；但是还会有内存泄漏存在，ThreadLocal被回收，key的值变成null，导致整个value再也无法被访问到：解决办法——在使用结束时，调用ThreadLocal.remove来释放其value的引用。

## **线程池状态**

线程池有5种状态：running, shutdown, stop, tidying, terminated

* running: 线程池处于运行状态，可以接受任务、执行任务，创建线程默认就是这个状态
* shutdown: 调用shutdown\(\)函数，不会接受新任务，但是会慢慢处理完堆积的任务
* stop: 调用shutdownNow\(\)函数，不会接受新任务，也不会处理已有的任务，会中断现有的任务
* tidying: 当线程池状态为showdown或者stop，任务数量为0，就会变为tidying。这个时候会调用钩子函数terminated
* terminated: terminated\(\)执行完成

在线程池中，用一个原子类来记录线程池的信息，用了int的高3位表示状态，后面的29位表示线程池中线程的数量。

## **Java线程池是如何实现的**

* 线程池中线程被抽象位内部静态类Worker，是基于AQS实现的，存放在HashSet中
* 要被执行的线程存放在BlockingQueue中
* 基本思想就是从workQueue中取出要执行的任务，放在worker中处理

## **如果线程池中的一个线程运行时出现了异常，会发生什么**

如果提交任务的时候使用了submit，那么返回的feature中会存有异常信息；如果是execute则会打印出异常栈，但是不会给其他线程造成影响。之后线程池会删除该线程，增加一个worker。

## **如果线程池中的一个线程运行时出现了异常，会发生什么**

* 提交一个任务，线程池里存活的核心线程数小于corePoolSize时，线程池会创建一个核心线程去处理提交的任务
* 如果线程池核心线程数已满，即线程数已经等于corePoolSize时，一个新提交的任务，会被放进任务队列workQueue排队等待执行
* 当线程池里面存活的线程数已经等于corePoolSize，并且任务队列workQueue也满时，判断线程数是否达到了maximumPoolSize，即最大线程数是否已满，如果没达到，创建非核心线程执行提交的任务。
* 如果当前的线程数达到了maximumPoolSize，还有新的任务过来的话，直接采用拒绝策略处理。

## **拒绝策略**

* AbortPolicy：直接抛出异常阻止线程运行
* CallerRunsPolicy：如果被抛弃的线程任务未关闭，则执行该线程
* DiscardOldestPolicy：移除队列最早线程，尝试提交当前任务
* DiscardPolicy：丢弃当前任务，不做处理

## **newFixedThreadPool（固定数目线程的线程池）**

* 阻塞队列为无界队列LinkedBlockingQueue
* 适用于处理CPU密集型的任务，使用执行长期的任务

## **newCachedThreadPool（可缓存线程的线程池）**

* 阻塞队列是SynchronousQueue
* 适用于并发执行大量短期的小任务

## **newSingleThreadExecutor（单线程的线程池）**

* 阻塞队列是LinkedBlockingQueue
* 适用于串行执行任务的场景，一个任务一个任务地执行

## **newScheduledThreadPool（定时及周期执行的线程池）**

* 阻塞队列是DelayedWorkQueue
* 周期性执行任务的场景，需要限制线程数量的场景

