---
title: tcp协议简述
date: 2020-04-23
next: false
---

## 特点
### 面向连接
tcp传输时会先建立tcp连接，然后传输数据，传输完毕后断开连接。
### 一对一全双工
支持单个发送方和接收方间的双向传输。

## 报文段结构
### 首部字段
源端口号|目的端口号
:-:|:-:
和多路复用、分解相关

序号|
:-:|
确认号|
实现可靠信息传输

首部长度|保留未用|标志字段|接收窗口
:-:|:-:|:-:|:-:|
实现可靠数据传输。
标志字段：CWR、ECE、URG、ACK、PSH、RST、SYN、FIN
接收窗口：显示接收端接收缓存可用空间

因特网检验和|紧急数据指针
:-:|:-:
检验和实现标志检测，紧急数据指针一般不用。

选项|
:-:|
调节最大报文长度、或作为窗口调节因子时使用。
### 数据字段
数据字段|
:-:|
应用层进程传入的报文数据

## 实现功能
### 可靠数据传输
发送端输入数据和接收端传出数据最终是完全相同的。
#### 为什么TCP需要实现可靠数据传输？
网络层不保证数据报的交付、数据报的按序交付、数据报的完整性。所以TCP需在应用层两端实现。
#### 实现原理
TCP的实现原理基于之前可靠数据传输协议的探索，包括差错检测、重传、累计确认、定时器及用于序号和确认号的首部字段，添加了改进。
#### 建立TCP连接的流程
1.应用层进程向socket发起传输请求，并传入数据。
2.**第一次握手**，建立连接开始。发送端向接收端发送SYN报文段（其中SYN标志位置为1），生 成初始序号C填入序号字段中。
3.**第二次握手**，接收端收到SYN报文段，提取其中信息，分配接收端缓存、变量。生成SYNACK报文段（SYN标志位置为 1，序号为C+1，确认号为接收端初始序号S），并发送给发送端。表明已收到发送端建立连接的报文段。
4.**第三次握手**发送端收到SYNASK报文段，分配发送端缓存、变量。发送另外报文段表明已收到接收端同意连接的报文段。报文段SYN标志位置为0，序号为S+1，可携带数据字段。连接建立。
#### 断开TCP连接的流程
客户端和服务端都可终止TCP连接，以客户端为例。
1.应用层进程发起终止TCP命令，客户端向服务端发送终止报文段（FIN标志位置为1）。此时，客户端状态变为finish_wait_1。
2.服务端接收报结束文段，返回确认报文。服务端状态变为closed_wait。客户端收到此报文段时状态变为finish_wait_2。
3.服务端确定没有要发送的数据后，向客户端发送终止报文段。服务端状态变为last_ack。
4.客户端收到终止报文段，返回确认报文，状态变为time_wait。
5.客户端等待一定时间变更为closed状态释放连接所需资源。服务端接受到确认报文段后状态立即变为closed状态。
### 拥塞控制
#### 为什么要实现拥塞控制
链路传输速率、路由缓存是有限的，链路中发送数据报速率过大时，会造成分组丢失（丢包）、时延增大。实现拥塞控制可以提升吞吐量（接收方接收数据速率）、减小时延。
#### 实现方案
##### 端到端拥塞控制
  网络层不向端系统提供显式拥塞信息，由端系统自行判断并解决。TCP采用此方案。
##### 网络辅助拥塞控制
网络层向端系统提供显式拥塞信息。具体有两种
- 直接向发送方反馈网络信息。
- 网络层标记出现拥塞的分组，接收方接收到通知发送方。
#### TCP所实现的拥塞控制
##### TCP为什么要实现拥塞控制
已清楚拥塞控制的必要性，但其他网络层级不一定实现了可靠的拥塞控制，所以在传输层自行实现。
##### 从三个方面入手实现
1.控制拥塞窗口（cwnd）的大小调节发送速率。
2.过度拥塞时，路径上的路由缓存溢出时，会造成数据报丢失，引发丢包事件（超时或收到3个冗余确认报文），表示出现了拥塞。
3.通过TCP拥塞控制算法调节拥塞窗口大小，可以提高链路利用率，又可消除拥塞。
#### TCP拥塞控制算法
    *涉及名词
      cwnd  拥塞窗口
      ssthresh 慢启动阈值
      duplicate ACK 接收到一个冗余确认
      dupACKCount 接收到冗余确认的个数
      MSS  最大报文长度*
##### 3个阶段
###### 1.慢启动
  初始阶段，此阶段内cwnd指数增加（2的n次方）。
  超时一次,ssthresh=cwnd/2,cwnd置为1。
  cwnd大于等于ssthresh时，进入拥塞避免阶段。
  接收3次冗余ACK进入快速恢复阶段。
###### 2.拥塞避免
  大多数时间处于此阶段，阶段内cwnd线性增长。故TCP也称为*加性增，乘性减*。
  接收到3此冗余ACK时进入快速恢复阶段，cwnd变为一半。
  超时时进入慢启动阶段，cwnd变为一半。
###### 3.快速恢复（非必须）
  接收到三次冗余ACK时，进入此阶段。
  接收到新的确认时回到拥塞避免阶段，cwnd=ssthresh。
  超时时进入慢启动，ssthresh为cwnd一半，cwnd置为1。
  期间接收到冗余ACK时，cwnd+=MSS
##### 阶段之间转化

![拥塞控制状态转移图](~@blogImg/tcp-crowd.png)