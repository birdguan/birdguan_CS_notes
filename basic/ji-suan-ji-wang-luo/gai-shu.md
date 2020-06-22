# 概述

## 网络的网络 <a id="%E7%BD%91%E7%BB%9C%E7%9A%84%E7%BD%91%E7%BB%9C"></a>

网络把主机连接起来，而互联网是把多种不同的网络连接起来，因此互联网是网络的网络。而互联网是全球范围的互联网。

## ISP <a id="ISP"></a>

互联网服务商ISP可以从互联网管理机构获得许多IP地址，同时拥有通信线路以及路由器等联网设备，个人或机构向ISP缴纳一定的费用就可以接入互联网。

目前的互联网是一种多层次ISP结构，ISP根据覆盖面积的大小分为第一层ISP、区域ISP和接入ISP。互联网交换点IXP允许两个ISP直接相连而不用经过第三个ISP。

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)​

![](https://img-blog.csdnimg.cn/20200427205306726.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMwMjQxNzA5,size_16,color_FFFFFF,t_70)

## 主机之间的通信方式 <a id="%E4%B8%BB%E6%9C%BA%E4%B9%8B%E9%97%B4%E7%9A%84%E9%80%9A%E4%BF%A1%E6%96%B9%E5%BC%8F"></a>

* 客户-服务器（C/S）：客户式服务的请求方，服务器是服务的提供方。
* 对等（P2P）：不区分客户和服务器。

## 电路交换与分组交换 <a id="%E7%94%B5%E8%B7%AF%E4%BA%A4%E6%8D%A2%E4%B8%8E%E5%88%86%E7%BB%84%E4%BA%A4%E6%8D%A2"></a>

### 1.电路交换 <a id="1.%E7%94%B5%E8%B7%AF%E4%BA%A4%E6%8D%A2"></a>

电路交换用于电话通信系统，两个用户要通信之前需要建立一条专用的物理链路，并且在整个通信过程中始终占用该链路。由于通信的过程中不可能一直在使用传输线路，因此电路交换对线路的利用率很低，往往不到10%。

### 2.分组交换 <a id="2.%E5%88%86%E7%BB%84%E4%BA%A4%E6%8D%A2"></a>

每个分组都有首部和尾部，包括了源地址和目的地址等控制信息，在同一个传输线路上同时传输多个分组互相不会影响，因此在同一条传输线路上允许同时传输多个分组，也就是说分组交换不需要占用传输线路。

在一个邮局通信系统中，邮局收到一份邮件之后，先存储下来，然后把相同目的地的邮件一起转发到下一个目的地，这个过程就是存储转发过程，分组交换也使用了存储转发过程。

## 时延 <a id="%E6%97%B6%E5%BB%B6"></a>

总时延=排队时延+处理时延+传输时延+传播时延

![](https://img-blog.csdnimg.cn/20200427205831676.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMwMjQxNzA5,size_16,color_FFFFFF,t_70)

### 1.排队时延 <a id="1.%E6%8E%92%E9%98%9F%E6%97%B6%E5%BB%B6"></a>

分组在路由器的输入队列和输出队列中排队等待的时间，取决于网络当前的通信量。

### 2.处理时延 <a id="2.%E5%A4%84%E7%90%86%E6%97%B6%E5%BB%B6"></a>

主机或路由器收到分组时进行处理所需要的时间，例如分析首部、从分组中提取数据、进行差错检验或查找适当的路由等。

### 3.传输时延 <a id="3.%E4%BC%A0%E8%BE%93%E6%97%B6%E5%BB%B6"></a>

主机或路由器传输数据帧所需要的时间。

$$
delay=\frac{l(bit)}{v(bit/s)}
$$

其中，l表示数据帧的长度，v表示传输速率。

### 4.传播时延 <a id="4.%E4%BC%A0%E6%92%AD%E6%97%B6%E5%BB%B6"></a>

电磁波在信道中传播所需要花费的时间，电磁波传播的速度接近光速。

## 计算机网络的体系结构 <a id="%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C%E7%9A%84%E4%BD%93%E7%B3%BB%E7%BB%93%E6%9E%84"></a>

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)​

![](https://img-blog.csdnimg.cn/20200427210207455.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMwMjQxNzA5,size_16,color_FFFFFF,t_70)

### 1.五层协议 <a id="1.%E4%BA%94%E5%B1%82%E5%8D%8F%E8%AE%AE"></a>

* **应用层：**为特定应用程序提供数据传输服务，例如HTTP、DNS等协议。数据单位为报文。
* **传输层：**为进程提供通用数据传输服务。由于应用层协议很多，定义通用的传输层协议就可以支持不断增多的应用层协议。运输层包括两种协议：传输控制协议TCP，提供面向连接的、可靠的数据传输服务，数据单位为报文段；用户数据报协议UDP，提供无连接、尽最大努力的数据传输服务，数据单位为用户数据报。TCP主要提供完整性服务，UDP主要提供及时性服务。
* **网络层：**为主机提供数据传输服务。而传输层协议是为主机中的进程提供数据传输服务。网络层把传输层传递下来的报文段后者用户数据封装成分组。
* **数据链路层：**网络层针对的还是主机之间的数据传输服务，而主机之间可以有很多链路，链路层协议就是为同一链路的主机提供数据传输服务。数据链路层把网络层传下来的分组封装成帧。
* **物理层：**考虑的是怎样在传输媒体上传输数据比特流，而不是指具体的传输媒体。物理层的作用是尽可能屏蔽传输媒体和通信手段的差异，使链路层感觉不到这些差异。

### 2.OSI <a id="2.OSI"></a>

其中表示层和会话层用途如下：

* **表示层：**数据压缩、加密以及数据描述，这使得应用程序不必关系在各台主机中数据内部格式不同的问题。
* **会话层：**建立及管理会话。

五层协议没有表示层和会话层，而是将这些功能留给应用程序开发者处理。

### 3.TCP/IP <a id="3.TCP%2FIP"></a>

它只有四层，相当于五层协议中数据链路层和物理层合并为网络接口层。

TCP/IP系统结构不严格遵循OSI分层概念，应用层可能会直接使用IP层或者网络接口层。

![](https://img-blog.csdnimg.cn/20200427211720762.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMwMjQxNzA5,size_16,color_FFFFFF,t_70)

### 4.数据在各层之间的传递过程 <a id="4.%E6%95%B0%E6%8D%AE%E5%9C%A8%E5%90%84%E5%B1%82%E4%B9%8B%E9%97%B4%E7%9A%84%E4%BC%A0%E9%80%92%E8%BF%87%E7%A8%8B"></a>

在向下的过程中，需要添加下层协议所需要的首部或者尾部，而在向上的过程中不断拆开首部和尾部。

路由器只有下面三层协议，因为路由器位于网络核心中，不需要为进程或者应用程序提供服务，因此也就不需要传输层和应用层。

