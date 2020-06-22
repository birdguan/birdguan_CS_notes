# 链路层

## 基本问题 <a id="%E5%9F%BA%E6%9C%AC%E9%97%AE%E9%A2%98"></a>

### 1.封装成帧 <a id="1.%E5%B0%81%E8%A3%85%E6%88%90%E5%B8%A7"></a>

将网络层传下来的分组添加首部和尾部，用于标记帧的开始和结束。

![](https://img-blog.csdnimg.cn/20200427212817370.png)

### 2.透明传输 <a id="2.%E9%80%8F%E6%98%8E%E4%BC%A0%E8%BE%93"></a>

透明表示一个实际存在是事物看起来好像不存在一样。

帧是用首部和尾部进行定界，如果帧的数据部分含有和首部尾部相同的内容，那么帧的开始和结束位置就会被错误地判定。需要在数据部分出现首部尾部相同的内容前面插入转义字符。如果数据部分出现转义字符，那么就在转义字符前面再加个转义字符。在接收端进行处理之后可以还原出原始数据。这个过程透明传输的内容是转义字符，用户察觉不到转义字符的存在。

![](https://img-blog.csdnimg.cn/20200427213127153.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMwMjQxNzA5,size_16,color_FFFFFF,t_70)

### 3.差错检测 <a id="3.%E5%B7%AE%E9%94%99%E6%A3%80%E6%B5%8B"></a>

目前数据链路层广泛使用**循环冗余检验（CRC）**来检查比特差错。

## 信道分类 <a id="%E4%BF%A1%E9%81%93%E5%88%86%E7%B1%BB"></a>

### 1.广播信道 <a id="1.%E5%B9%BF%E6%92%AD%E4%BF%A1%E9%81%93"></a>

一对多通信，一个节点发送的数据能够被广播信道上的所有节点接收到。

所有的节点都在同一个广播信道上发送数据，因此需要有专门的控制方法进行协调，避免发生冲突（冲突也叫碰撞）。

主要有两种控制方法进行协调，一个是使用信道复用技术，一是使用CSMA/CD协议。

### 2.点对点信道 <a id="2.%E7%82%B9%E5%AF%B9%E7%82%B9%E4%BF%A1%E9%81%93"></a>

一对一通信。

因为不会发生碰撞，因此也比较简单，使用PPP协议进行控制。

## 信道复用技术 <a id="%E4%BF%A1%E9%81%93%E5%A4%8D%E7%94%A8%E6%8A%80%E6%9C%AF"></a>

### 1.频分复用 <a id="1.%E9%A2%91%E5%88%86%E5%A4%8D%E7%94%A8"></a>

频分复用的所有主机在相同的时间占有不同的频率带宽资源。

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)​

![](https://img-blog.csdnimg.cn/20200427213543280.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMwMjQxNzA5,size_16,color_FFFFFF,t_70)

### 2.时分复用 <a id="2.%E6%97%B6%E5%88%86%E5%A4%8D%E7%94%A8"></a>

时分复用的主机在不同的时间占用相同的频率带宽资源。

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)​

![](https://img-blog.csdnimg.cn/2020042721362910.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMwMjQxNzA5,size_16,color_FFFFFF,t_70)

 使用平分复用和时分复用进行通信，在通信的过程中主机会一直占有一部分信道资源。但是由于计算机数据的突发性质，通信过程中没必要一直占用信道资源不让出给其它用户使用，因此这两种方式对信道的利用率都不高。

### 3.统计时分复用 <a id="3.%E7%BB%9F%E8%AE%A1%E6%97%B6%E5%88%86%E5%A4%8D%E7%94%A8"></a>

是对时分复用的一种改进，不固定每个用于在时分复用帧中的位置，只要有数据就集中起来组成统计时分复用帧然后发送。

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)​

![](https://img-blog.csdnimg.cn/20200427213932740.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMwMjQxNzA5,size_16,color_FFFFFF,t_70)

### 4.波分复用 <a id="4.%E6%B3%A2%E5%88%86%E5%A4%8D%E7%94%A8"></a>

光的频分复用，由于光的频率很高，因此习惯上用波长而不是频率来表示所使用的光载波。

### 5.码分复用 <a id="5.%E7%A0%81%E5%88%86%E5%A4%8D%E7%94%A8"></a>

为每个用户分配m比特的码片，并且所有的码片正交，对于任意两个码片![\overrightarrow{S}](https://private.codecogs.com/gif.latex?%5Coverrightarrow%7BS%7D)和![\overrightarrow{T}](https://private.codecogs.com/gif.latex?%5Coverrightarrow%7BT%7D)有：

$$
\frac{1}{m}\overrightarrow{S}\cdot \overrightarrow{T}=0
$$

为了讨论方便，取m=8，设码片![\overrightarrow{S}](https://private.codecogs.com/gif.latex?%5Coverrightarrow%7BS%7D)为0001 1011.拥有该码片的用户发送比特1时就发送该码片，发送比特0时就发送改密pain的反码1110 0100。

在计算时将0001 1011记作-1-1-1+1 +1-1+1+1，可以得到

$$
\frac{1}{m}\overrightarrow{S}\cdot \overrightarrow{S}=1
$$

$$
\frac{1}{m}\overrightarrow{S}\cdot \overrightarrow{S}'=-1
$$

其中![\overrightarrow{S}&apos;](https://private.codecogs.com/gif.latex?%5Coverrightarrow%7BS%7D%27)为![\overrightarrow{S}](https://private.codecogs.com/gif.latex?%5Coverrightarrow%7BS%7D)的反码。

利用上面的式子我们直到，当接收端使用码片![\overrightarrow{S}](https://private.codecogs.com/gif.latex?%5Coverrightarrow%7BS%7D)对接收到的数据进行内积运算时，结果为0的是其它用户发送的数据，结果为1的使用户发送的比特1，结果为-1的是用户发送的比特0.

码分复用需要发送的数据量是原先的m倍。

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)​

![](https://img-blog.csdnimg.cn/20200427214759563.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMwMjQxNzA5,size_16,color_FFFFFF,t_70)

## CSMA/CD协议 <a id="CSMA%2FCD%E5%8D%8F%E8%AE%AE"></a>

CSMA/CD表示载波监听多点接入/碰撞检测。

* **多点接入：**说明这是总线型网络，许多主机以多点的方式连接到总线上。
* **载波监听：**每个主机都必须不停地监听信道。在发送前，如果监听到信道正在使用，就必须等待。
* **碰撞检测：**在发送中，如果监听到信道已有其他主机正在发送数据，就表示发生了碰撞。虽然每个主机在发送数据之前都已经监听到信道为空闲，但是由于电磁波传播时延的存在，还是有可能会发生碰撞。

记端到端的传播时延为![\tau](https://private.codecogs.com/gif.latex?%5Ctau)，最先发送的站点最多经过![2\tau](https://private.codecogs.com/gif.latex?2%5Ctau)就可以直到是否发生了碰撞，称![2\tau](https://private.codecogs.com/gif.latex?2%5Ctau)为争用期。只有经过争用期之后还没有检测到碰撞，才能肯定这次发送不会发生碰撞。

当发生碰撞时，站点要停止发送，等待一段时间再发送。这个时间采用截断二进制指数退避算法来确定。从离散的整数集合![\{0,1, \cdots , \(2^k-1\)\}](https://private.codecogs.com/gif.latex?%5C%7B0%2C1%2C%20%5Ccdots%20%2C%20%282%5Ek-1%29%5C%7D)中随机取出一个数，记作r，然后取r倍的争用期作为重传等待时间。

![](https://img-blog.csdnimg.cn/20200427215631361.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMwMjQxNzA5,size_16,color_FFFFFF,t_70)

## PPP协议 <a id="PPP%E5%8D%8F%E8%AE%AE"></a>

互联网用户通常需要连接某个ISP之后才能接入到互联网，PPP协议是用户计算机和ISP进行通信时所使用的数据链路层协议。

![](https://img-blog.csdnimg.cn/20200427215836140.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMwMjQxNzA5,size_16,color_FFFFFF,t_70)

 PPP的帧格式：

* F字段为帧的定界符
* A和C字段暂时没有意义
* FCS字段是使用CRC的检测序列
* 信息部分的长度不超过1500

![](https://img-blog.csdnimg.cn/20200427220011297.png)

## MAC地址 <a id="MAC%E5%9C%B0%E5%9D%80"></a>

MAC地址是链路层地址，长度为6字节（48位）,用于唯一标识网络适配器（网卡）。

一台主机拥有多少个网络适配器就用有多少个MAC地址。例如笔记本普遍存在无线网络适配器和有线网络适配器，因此有两个MAC地址。

## 局域网 <a id="%E5%B1%80%E5%9F%9F%E7%BD%91"></a>

局域网是一种典型的广播信道，主要特点是网络位一个单位所拥有，且地址范围和站点数目均有限。

主要有以太网、令牌环网、FDDI和ATM等局域网技术，目前以太网占领着有线局域网市场。

可以按照网络拓扑结构对局域网进行分类：

![](https://img-blog.csdnimg.cn/20200427220338895.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMwMjQxNzA5,size_16,color_FFFFFF,t_70)

## 以太网 <a id="%E4%BB%A5%E5%A4%AA%E7%BD%91"></a>

以太网是一种星型拓扑结构局域网。

早期使用集线器进行连接，集线器是一种物理层设备，作用于比特而不是帧，当一个比特到达接口时，集线器重新生成这个比特，并将其能量强度放大，从而扩大网络的传输距离，之后再将这个比特发送到其它所有接口。如果集线器同时接收到两个不同接口的帧，那么就发生了碰撞。

目前以太网使用交换机代替了集线器，交换机是一种链路层设备，他不会发生碰撞，能根据MAC地址进行存储转发。

以太网帧格式：

* 类型：标记上层使用的协议；
* 数据：长度在46-1500之间，如果太小则需要填充；
* FCS：帧检验序列，使用的是CRC检测方法

![](https://img-blog.csdnimg.cn/20200427221840259.png)

## 交换机 <a id="%E4%BA%A4%E6%8D%A2%E6%9C%BA"></a>

交换机具有自学习能力，学习的是交换表的内容，交换表中存储着MAC地址到接口的映射。

正是由于这种自学能力，因此交换机是一种即插即用设备，不需要网络管理员手动配置交换表内容。

下图中，交换机有4个接口，主机A向主机B发送数据帧时，交换机把主机A到接口1的映射写入交换表中。为了发送数据帧到B，先查交换表，此时没有主机B的表项，那么主机A就发送广播帧，主机C和主机D就会丢弃该帧，主机B回应该帧向主机A发送数据包时，交换机查找主机A映射的接口为1，就发送数据帧到接口1，同时交换机添加主机B到接口2的映射。

![](https://img-blog.csdnimg.cn/20200427222531524.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMwMjQxNzA5,size_16,color_FFFFFF,t_70)

## 虚拟局域网 <a id="%E8%99%9A%E6%8B%9F%E5%B1%80%E5%9F%9F%E7%BD%91"></a>

虚拟局域网可以建立与物理无关的逻辑组，只有在同一个局域网中的成员才会收到链路层广播信息。

例如下图中（A1, A2, A3, A4）属于一个虚拟局域网，A1发送的广播会被A2, A3, A4收到，而其他站点收不到。

使用VLAN干线连接起来建立虚拟局域网，每台交换机上的一个特殊接口被设置为干线接口，以连接VLAN交换机。IEEE定义了一种扩展的以太网帧格式802.1Q，它在标准以太网帧上加入了4字节首部VLAN标签，用于标识该帧属于哪一个虚拟局域网。

![](https://img-blog.csdnimg.cn/20200427222924624.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMwMjQxNzA5,size_16,color_FFFFFF,t_70)

