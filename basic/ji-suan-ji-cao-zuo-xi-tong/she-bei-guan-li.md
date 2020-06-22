# 设备管理

## 磁盘管理 <a id="%E7%A3%81%E7%9B%98%E7%AE%A1%E7%90%86"></a>

* 盘面（Platter）：一个磁盘有多个盘面
* 磁道（Track）：盘面上的圆形带状区域，一个盘面可以有多个磁道
* 扇区（Track Selector）：磁道上的一个弧段，一个磁道可以有多个扇区，它是最小的物理存储单位，目前主要有512bytes和4k两种大小
* 磁头（Head）：与盘面非常接近，能够将盘面上的磁场转换为电信号，或者将电信号转换为盘面的磁场
* 制动手臂（Actuator arm）：用于在磁道之间移动磁头
* 主轴（Spindle）：使整个盘面转动

![](../../.gitbook/assets/image%20%2848%29.png)

## 磁盘调度算法 <a id="%E7%A3%81%E7%9B%98%E8%B0%83%E5%BA%A6%E7%AE%97%E6%B3%95"></a>

读写一个磁盘块的时间的影响因素有：

* 旋转时间
* 寻道时间
* 实际的数据传输时间

### 1.先来先服务 First Come First Served <a id="1.%E5%85%88%E6%9D%A5%E5%85%88%E6%9C%8D%E5%8A%A1"></a>

按照磁盘请求的顺序进行调度。

优点是公平和简单。缺点也很明显，因为未对寻道做任何优化，使平均寻道时间可能较长。

### 2.最短寻道时间优先 Shortest Seek Time First <a id="2.%E6%9C%80%E7%9F%AD%E5%AF%BB%E9%81%93%E6%97%B6%E9%97%B4%E4%BC%98%E5%85%88"></a>

优先调度与当前磁头所在磁道距离最近的磁道。

虽然寻道时间比较低，但是不够公平。如果新到达的磁道请求总是比一个在等到的磁道请求近，那么在等待的磁道请求就会一直等待下去，也就是会出现饥饿现象。具体来说，两端的磁道请求更容易出现饥饿现象。

![](../../.gitbook/assets/image%20%2840%29.png)

### 3.电梯算法 SCAN <a id="3.%E7%94%B5%E6%A2%AF%E7%AE%97%E6%B3%95"></a>

电梯总是保持一个方向运行，直到该方向没有请求为止，然后改变运行方向。

电梯算法（扫描算法）和电梯运行过程类似，总是按一个方向来进行磁盘调度，知道该方向上没有未完成的磁盘请求，然后改变方向。

因为考虑了移动方向，因此所有的磁盘请求都会被满足，解决了SSTF的饥饿问题。

![](../../.gitbook/assets/image%20%2852%29.png)
