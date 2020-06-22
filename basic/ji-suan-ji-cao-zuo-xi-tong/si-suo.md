# 死锁

## 死锁必要条件 <a id="%E6%AD%BB%E9%94%81%E5%BF%85%E8%A6%81%E6%9D%A1%E4%BB%B6"></a>

1. 互斥：每个资源要么已经分配给了一个进程，要么就是可用的。
2. 占有和等待：已经得到了某个资源的进程可以再请求新的资源。
3. 不可抢占：已经分配给一个进程的资源不能强制性地被抢占，它只能被占用它地进程显式地释放。
4. 环路等待：有两个或者两个以上的进程组成一条环路，该环路中的每个进程都在等待下一个进程所占用的资源。

## 处理方法 <a id="%E5%A4%84%E7%90%86%E6%96%B9%E6%B3%95"></a>

主要有以下四个方法：

1. 鸵鸟策略
2. 死锁检测与死锁恢复
3. 死锁预防
4. 死锁避免

## 鸵鸟策略 <a id="%E9%B8%B5%E9%B8%9F%E7%AD%96%E7%95%A5"></a>

把头埋在沙子里，假装根本没发生问题。

因为解锁死锁问题的代价很高，因此鸵鸟策略这种不采取任何措施的方案会获得更高的性能。

当发生死锁时不会对用户造成多大损失，或发生死锁的概率很低，可以采用鸵鸟策略。

大多数操作系统，包括Unix，Linux和Win，处理死锁问题的办法仅仅是忽略它。

## 死锁检测与死锁恢复 <a id="%E6%AD%BB%E9%94%81%E6%A3%80%E6%B5%8B%E4%B8%8E%E6%AD%BB%E9%94%81%E6%81%A2%E5%A4%8D"></a>

不试图阻止死锁，而是当检测到死锁发生时，采取措施进行恢复。

### 1.每种类型一个资源的死锁检测 <a id="1.%E6%AF%8F%E7%A7%8D%E7%B1%BB%E5%9E%8B%E4%B8%80%E4%B8%AA%E8%B5%84%E6%BA%90%E7%9A%84%E6%AD%BB%E9%94%81%E6%A3%80%E6%B5%8B"></a>

![](../../.gitbook/assets/image%20%2868%29.png)

上图为资源分配图，其中方框表示资源，圆圈表示进程。资源指向进程表示该资源已经分配给该进程，进程指向资源表示进程请求获取该资源。

图a可以取出环，如图b所示，它满足了环路等待条件，因此会发生死锁。

每种类型一个资源的死锁检测算法是通过检测有向图是否存在环来实现，从一个节点出发进行深度优先搜索，对访问过的节点进行标记，如果访问了已经标记的节点，就表示有向图存在环，也就是检测到了死锁的发生。

### 2.每种类型多个资源的死锁检测 <a id="2.%E6%AF%8F%E7%A7%8D%E7%B1%BB%E5%9E%8B%E5%A4%9A%E4%B8%AA%E8%B5%84%E6%BA%90%E7%9A%84%E6%AD%BB%E9%94%81%E6%A3%80%E6%B5%8B"></a>

![](../../.gitbook/assets/image%20%2836%29.png)

上图中，有三个进程四个资源，每个数据代表的含义如下：

* E向量：资源总量
* A向量：资源剩余量
* C矩阵：每个进程所拥有的资源数量，每一行都代表一个进程拥有资源的数量
* R矩阵：每个进程请求的资源数量

进程P1和P2所请求的资源都得不到满足，只有进程P3可以，让P3执行，之后释放P3所拥有的资源，此时A=\(2 2 2 0\)。P2可以执行，执行后释放P2拥有的资源，A=\(4 2 2 1\)。P1也可以执行。所有的进程都可以顺利执行，没有死锁。

算法总结如下：

每个进程最开始都不被标记，执行过程有可能被标记。当算法结束时，任何没有被标记的进程都是死锁进程。

1. 寻找一个没有被标记的进程Pi，它所请求的资源小于等于A。
2. 如果找到了这样一个进程，那么将C矩阵的第i行向量加到A中，标记该进程，并转回1。
3. 如果没有这样一个进程，算法终止。

### 3.死锁恢复 <a id="3.%E6%AD%BB%E9%94%81%E6%81%A2%E5%A4%8D"></a>

* 利用抢占恢复
* 利用回滚恢复
* 通过杀死进程恢复

## 死锁预防 <a id="%E6%AD%BB%E9%94%81%E9%A2%84%E9%98%B2"></a>

在程序运行之前预防发生死锁。

### 1.破坏互斥条件 <a id="1.%E7%A0%B4%E5%9D%8F%E4%BA%92%E6%96%A5%E6%9D%A1%E4%BB%B6"></a>

例如假脱打印机技术允许若干个进程同时输出，唯一真正请求物理打印机的进程是打印机守护进程。

### 2.破坏占有和等待条件 <a id="2.%E7%A0%B4%E5%9D%8F%E5%8D%A0%E6%9C%89%E5%92%8C%E7%AD%89%E5%BE%85%E6%9D%A1%E4%BB%B6"></a>

一种实现方式是规定所有进程在开始执行前请求所需要的全部资源。

### 3.破坏不可抢占条件 <a id="3.%E7%A0%B4%E5%9D%8F%E4%B8%8D%E5%8F%AF%E6%8A%A2%E5%8D%A0%E6%9D%A1%E4%BB%B6"></a>

### 4.破坏环路等待条件 <a id="4.%E7%A0%B4%E5%9D%8F%E7%8E%AF%E8%B7%AF%E7%AD%89%E5%BE%85%E6%9D%A1%E4%BB%B6"></a>

给资源统一编号，进程只能按照编号顺序来请求资源。

## 死锁避免 <a id="%E6%AD%BB%E9%94%81%E9%81%BF%E5%85%8D"></a>

在程序运行时避免发生死锁。

### 1.安全状态 <a id="1.%E5%AE%89%E5%85%A8%E7%8A%B6%E6%80%81"></a>

![](../../.gitbook/assets/image%20%2821%29.png)

图a的第二列Has表示已经拥有的资源数，第三列Max表示总共需要的资源数，Free表示还有可以使用的资源数。从图a开始出发，先让B拥有所需的所有资源（图b\)，运行结束后释放B，此时Free变成5（图c）；接着以同样的方式运行C和A，使得所有的进程都能成功运行，因此可以称图a所示的状态是安全的。

定义：如果没有死锁发生，并且即使所有进程突然请求对资源的最大需求，也仍然存在某种调度次序能够使得每一个进程运行完毕，则称该状态是安全的。

安全状态的检测与死锁的检测类似，因为安全状态必须要求不能发生死锁。下面的银行家算法与死锁检测算法非常相似。

### 2.单个资源的银行家算法 <a id="2.%E5%8D%95%E4%B8%AA%E8%B5%84%E6%BA%90%E7%9A%84%E9%93%B6%E8%A1%8C%E5%AE%B6%E7%AE%97%E6%B3%95"></a>

一个小城镇的银行家，他向一群客户分别承诺了一定的贷款额度，算法要做的是判断对请求的满足是否会进入不安全状态，如果是，就拒绝请求；否则予以分配。

![](../../.gitbook/assets/image%20%2896%29.png)

以上图c为不安全状态，因此算法会拒绝之前的请求，从而避免进入图c中的状态。 

### 3.多个资源的银行家算法 <a id="3.%E5%A4%9A%E4%B8%AA%E8%B5%84%E6%BA%90%E7%9A%84%E9%93%B6%E8%A1%8C%E5%AE%B6%E7%AE%97%E6%B3%95"></a>

![](../../.gitbook/assets/image%20%2894%29.png)

 上图中有五个进程，四个资源。左边的图表示已经分配的资源，右边的图便是还需要分配的资源。最右边E, P以及A分别表示：总资源、已分配资源以及可用资源，以向量形式表示。

检查一个状态是否安全的算法如下：

* 查找右边的矩阵是否存在一行小于等于向量A。如果不存在这样的行，那么系统将会发生死锁，状态是不安全的。
* 假若找到这样一行，将该进程标记为终止，并将其已分配资源加到A中。
* 重复以上两步，指导所有进程都标记为终止，则状态时安全的。
