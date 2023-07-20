---
title: 计算机网络
date: 2022-06-20 19:35:13
tags:
- Courses
categories: 
- Courses
mathjax: true
description: 计算机专业课《计算机网络》笔记。主讲教师：申彦明。
top_img: 
cover: /image/ComputerNetwork/cover.png
---

# 计算机网络概述

1. 协议：定义了格式、网络实体发送、接收的顺序以及信息传送、接收采取的动作。

2. 网络核心

   1. 分组交换

      - 数据包传输延迟：$\dfrac{L}{R}s$
      - 存储转发：数据包必须全部到达路由器才能继续传输

      排队时延：路由器填满后会丢包

   2. 电路交换：专用资源、保证性能，常用于传统电话网络

      - 频分复用、时分复用

3. 丢失、时延、吞吐率

   1. 四种时延：处理时延、排队时延、传输时延（$L/R$）（包发送）、传播时延（$d/S$）（包在链路传输）
   2. 流量强度：$\dfrac{L*a}{R}$，其中R为带宽，a为包平均到达速率，L为包的长度
   3. 丢包：队列缓冲区优先，到达满队列的包被丢弃
   4. 吞吐量：从发送方到接收方的吞吐量(bit/time)（短板效应）

# 应用层

## 应用层概述

<img src="/image/ComputerNetwork/image-20220620144446452.png" alt="image-20220620144446452" style="zoom:60%;" />

## HTTP&Web

<img src="/image/ComputerNetwork/image-20220620144613034.png" alt="image-20220620144613034" style="zoom:70%;" />

![image-20220620144733127](/image/ComputerNetwork/image-20220620144733127.png)

<img src="/image/ComputerNetwork/image-20220620144902373.png" alt="image-20220620144902373" style="zoom:60%;" />

## DNS服务器

<img src="/image/ComputerNetwork/image-20220620145016636.png" alt="image-20220620145016636" style="zoom:67%;" />

<img src="/image/ComputerNetwork/image-20220620145135714.png" alt="image-20220620145135714" style="zoom:67%;" />

<img src="/image/ComputerNetwork/image-20220620145211848.png" alt="image-20220620145211848" style="zoom:67%;" />

<img src="/image/ComputerNetwork/image-20220620145236568.png" alt="image-20220620145236568" style="zoom:67%;" />

## SMTP

<img src="/image/ComputerNetwork/image-20220620145255564.png" alt="image-20220620145255564" style="zoom:80%;" />

# 传输层

## 概述

<img src="/image/ComputerNetwork/image-20220620152959121.png" alt="image-20220620152959121" style="zoom:80%;" />

## UDP

<img src="/image/ComputerNetwork/image-20220620153247681.png" alt="image-20220620153247681" style="zoom:80%;" />

## 可靠数据传输

- rdt2.0：停止等待—发送以后接收反馈，如果是NAK则再次发送，如果是ACK则等待下次发送。

<img src="/image/ComputerNetwork/image-20220620154810062.png" alt="image-20220620154810062" style="zoom:80%;" />

- rdt2.1：处理乱码的ACK和NAK：增加两个状态号

  <img src="/image/ComputerNetwork/image-20220620154901324.png" alt="image-20220620154901324" style="zoom:80%;" />

- rdt2.2：进行改进，取消了NAK，在ACK中加入ACK了哪个分组

  <img src="/image/ComputerNetwork/image-20220620155118940.png" alt="image-20220620155118940" style="zoom:80%;" />

  

- rdt3.0：比特差错和丢包同时存在：引入定时器，等待合理时间重传

  <img src="/image/ComputerNetwork/image-20220620155554234.png" alt="image-20220620155554234" style="zoom:80%;" />

- 回退N步：注意，接收方只维持当前期望的ACK序号，乱序到达的直接丢弃。

<img src="/image/ComputerNetwork/image-20220620155922128.png" alt="image-20220620155922128" style="zoom:80%;" />

- 选择重传：注意，每个分组均有定时器

  <img src="/image/ComputerNetwork/image-20220620160055780.png" alt="image-20220620160055780" style="zoom:80%;" />

  <img src="/image/ComputerNetwork/image-20220620160158224.png" alt="image-20220620160158224" style="zoom:80%;" />

## TCP

<img src="/image/ComputerNetwork/image-20220620160338270.png" alt="image-20220620160338270" style="zoom:80%;" />

<img src="/image/ComputerNetwork/image-20220620160505485.png" alt="image-20220620160505485" style="zoom:80%;" />

<img src="/image/ComputerNetwork/image-20220620160541231.png" alt="image-20220620160541231" style="zoom:80%;" />

<img src="/image/ComputerNetwork/image-20220620160613925.png" alt="image-20220620160613925" style="zoom:80%;" />

<img src="/image/ComputerNetwork/image-20220620160623145.png" alt="image-20220620160623145" style="zoom:80%;" />

![image-20220620160838797](/image/ComputerNetwork/image-20220620160838797.png)

## TCP拥塞控制原理

<img src="/image/ComputerNetwork/image-20220620160946683.png" alt="image-20220620160946683" style="zoom:80%;" />

![image-20220620161002885](/image/ComputerNetwork/image-20220620161002885.png)

- TCP拥塞控制和流量控制对比
  - 流量控制原因： TCP接收方和发送方都有一个发送缓存和接收缓存，如果接收方接收缓存满了，但是发送方还继续发送数据，就会导致数据溢出，丢包产生。由接收方告诉发送方。
  - 拥塞控制原因：因为网络拥塞采取的手段，是发送方自己感知的。

## 网络层——控制平面

<img src="/image/ComputerNetwork/image-20220620161613729.png" alt="image-20220620161613729" style="zoom:80%;" />

<img src="/image/ComputerNetwork/image-20220620161724415.png" alt="image-20220620161724415" style="zoom:80%;" />

<img src="/image/ComputerNetwork/image-20220620162051143.png" alt="image-20220620162051143" style="zoom:80%;" />

## IPV4

<img src="/image/ComputerNetwork/image-20220620162249242.png" alt="image-20220620162249242" style="zoom:80%;" />

<img src="/image/ComputerNetwork/image-20220620162329833.png" alt="image-20220620162329833" style="zoom:80%;" />

<img src="/image/ComputerNetwork/image-20220620162411416.png" alt="image-20220620162411416" style="zoom:80%;" />

<img src="/image/ComputerNetwork/image-20220620162710544.png" alt="image-20220620162710544" style="zoom:80%;" />

## NAT

<img src="/image/ComputerNetwork/image-20220620162851604.png" alt="image-20220620162851604" style="zoom:80%;" />

## IPV6

<img src="/image/ComputerNetwork/image-20220620163200121.png" alt="image-20220620163200121" style="zoom:80%;" />

## SDN

<img src="/image/ComputerNetwork/image-20220620163257176.png" alt="image-20220620163257176" style="zoom:80%;" />

# 网络层——控制平面

## 概述

<img src="/image/ComputerNetwork/image-20220620164041753.png" alt="image-20220620164041753" style="zoom:80%;" />

## 路由选择算法

<img src="/image/ComputerNetwork/image-20220620164155045.png" alt="image-20220620164155045" style="zoom:80%;" />

<img src="/image/ComputerNetwork/image-20220620164234558.png" alt="image-20220620164234558" style="zoom:80%;" />

## AS内部选择——RIP

利用**跳数**来作为到达某个网络的路由选择标准。

1. 主要适用于规模较小的网络（当跳数为16时表示目的网络不可达）、可靠性要求较低的网络，可以通过不断的交换信息让路由器动态的适应网络连接的变化，这些信息包括每个路由器可以到达哪些网络，这些网络有多远等。
2. 当前路由器仅和相邻的路由器进行信息交换。
3. RIP更新过程
   1. 刚开始每个路由器只知道到直接连接网络的距离(距离为1)，接着每个路由器会和它的邻居路由器进行交换信息，更新自己的路由表
   2. 当路由器1与它的邻居路由器2进行信息沟通时，会把自己的路由表封装成数据报发给路由器2，路由器2接收到数据报后，会根据**对方的路由表中的信息**和**距离向量算法**来更新自己的路由表项。
   3. 经过若干次更新后，所有的路由器都会知道在本自治系统中任何一个网络的最短距离和下一跳的路由器地址。即达到收敛状态。
4. RIP缺点
   1. 只适用于小规模网络，因为当目的网络的距离(跳数)大于等于16时，路由器认为网络不可达。
   2. RIP协议在进行消息分享时，是把路由表中的所有表项都封装层RIP报文发送出去，占用的带宽比较大。
   3. Good news travel fast and bad news travel slow.
   4. RIP使用“跳数”作为最优距离并非总是最优路径

## AS内部选择——OSPF

对比于RIP，OSPF以**链路**状态为比较数值。

<img src="/image/ComputerNetwork/image-20220620164451917.png" alt="image-20220620164451917" style="zoom:80%;" />

<img src="/image/ComputerNetwork/image-20220620164600853.png" alt="image-20220620164600853" style="zoom:80%;" />

## AS之间选择——BGP

<img src="/image/ComputerNetwork/image-20220620165536630.png" alt="image-20220620165536630" style="zoom:80%;" />

- 热土豆：先在自己AS内部选取代价最小的，先让其离开自己的AS。

## SDN控制平面

<img src="/image/ComputerNetwork/image-20220620165842428.png" alt="image-20220620165842428" style="zoom:80%;" />

- OpenFlow协议

  <img src="/image/ComputerNetwork/image-20220620170542879.png" alt="image-20220620170542879" style="zoom:80%;" />

## ICMP——因特网控制报文协议

<img src="/image/ComputerNetwork/image-20220620170704682.png" alt="image-20220620170704682" style="zoom:80%;" />

- TTL用尽时，路由器向主机返回名称及IP地址指示本次传输失败。

## 网络管理协议和SNMP

- 使用UDP进行传输！

  <img src="/image/ComputerNetwork/image-20220620170824153.png" alt="image-20220620170824153" style="zoom:80%;" />

# 链路层和局域网

## 概述

<img src="/image/ComputerNetwork/image-20220620182502877.png" alt="image-20220620182502877" style="zoom:80%;" />

- TCP和链路层都提供了可靠数据传输，是否重复？

  - 不重复，数据帧在链路层传播时，链路层保证了主机到路由器或路由器到路由器之间传播无差错，而在路由器中对IP首部改变、对数据修改等操作都需要由TCP进行检测，链路层无法检测。

  - TCP

    <img src="/image/ComputerNetwork/image-20220620182857448.png" alt="image-20220620182857448" style="zoom:80%;" />

  - 链路层

    <img src="/image/ComputerNetwork/image-20220620182913371.png" alt="image-20220620182913371" style="zoom:80%;" />

## 差错检测——CRC校验码

见计算机组成原理部分。

## 多路访问链路及协议

- 协议要求：分布式、节点间交互必须使用此信道

  <img src="/image/ComputerNetwork/image-20220620183334094.png" alt="image-20220620183334094" style="zoom:80%;" />

  <img src="/image/ComputerNetwork/image-20220620183404704.png" alt="image-20220620183404704" style="zoom:80%;" />

  <img src="/image/ComputerNetwork/image-20220620183509647.png" alt="image-20220620183509647" style="zoom:80%;" />

  <img src="/image/ComputerNetwork/image-20220620183845739.png" alt="image-20220620183845739" style="zoom:80%;" />

## 交换局域网

- MAC地址作用：用于标识一个帧从哪个端口发出，到达哪个物理相连的其他端口。

  注意：ARP是网络层协议。

  <img src="/image/ComputerNetwork/image-20220620184859268.png" alt="image-20220620184859268" style="zoom:80%;" />

  <img src="/image/ComputerNetwork/image-20220620185128296.png" alt="image-20220620185128296" style="zoom:80%;" />

## 以太网

<img src="/image/ComputerNetwork/image-20220620185205698.png" alt="image-20220620185205698" style="zoom:80%;" />

## 交换机

<img src="/image/ComputerNetwork/image-20220620185706740.png" alt="image-20220620185706740" style="zoom:80%;" />

<img src="/image/ComputerNetwork/image-20220620185906475.png" alt="image-20220620185906475" style="zoom:80%;" />

## 路由器与交换机比较

1. 相同点

   1. 存储转发——缓存并转发，路由器基于IP地址转发，而交换机通过MAC地址进行转发。
   2. 均有转发表——路由器转发表通过路由算法生成，交换机转发表通过自学习生成。

2. 不同点

   1. 交换机更快，因为只到链路层；交换机更慢，因为实现到网络层

   2. 规模：交换机缺点：当规模大时，要浪费大量带宽资源，广播时对所有端口进行转发，占用大量资源。

   3. 广播域：

      <img src="/image/ComputerNetwork/image-20220620190332128.png" alt="image-20220620190332128" style="zoom:80%;" />

      <img src="/image/ComputerNetwork/image-20220620190514542.png" alt="image-20220620190514542" style="zoom:80%;" />

## 虚拟局域网

对物理设备进行虚拟划分。

<img src="/image/ComputerNetwork/image-20220620190644200.png" alt="image-20220620190644200" style="zoom:80%;" />

# 无线网络

## 概述

<img src="/image/ComputerNetwork/image-20220620191221002.png" alt="image-20220620191221002" style="zoom:80%;" />

## 无线信道

<img src="/image/ComputerNetwork/image-20220620191315488.png" alt="image-20220620191315488" style="zoom:80%;" />

## 802.11

<img src="/image/ComputerNetwork/image-20220620191436906.png" alt="image-20220620191436906" style="zoom:80%;" />

<img src="/image/ComputerNetwork/image-20220620191637632.png" alt="image-20220620191637632" style="zoom:80%;" />

<img src="/image/ComputerNetwork/image-20220620191900544.png" alt="image-20220620191900544" style="zoom:80%;" />

<img src="/image/ComputerNetwork/image-20220620191932022.png" alt="image-20220620191932022" style="zoom:80%;" />

<img src="/image/ComputerNetwork/image-20220620192554268.png" alt="image-20220620192554268" style="zoom:80%;" />

## 其他网络

<img src="/image/ComputerNetwork/image-20220620192646075.png" alt="image-20220620192646075" style="zoom:80%;" />

<img src="/image/ComputerNetwork/image-20220620192827477.png" alt="image-20220620192827477" style="zoom:80%;" />

<img src="/image/ComputerNetwork/image-20220620192856465.png" alt="image-20220620192856465" style="zoom:80%;" />

## 移动管理

<img src="/image/ComputerNetwork/image-20220620193213706.png" alt="image-20220620193213706" style="zoom:80%;" />

<img src="/image/ComputerNetwork/image-20220620193332991.png" alt="image-20220620193332991" style="zoom:80%;" />

<img src="/image/ComputerNetwork/image-20220620193438907.png" alt="image-20220620193438907" style="zoom:80%;" />

# 附录
<img src="/image/ComputerNetwork/1.jpg" alt="1" style="zoom:100%;" />
<img src="/image/ComputerNetwork/2.jpg" alt="2" style="zoom:100%;" />
<img src="/image/ComputerNetwork/3.jpg" alt="3" style="zoom:100%;" />
<img src="/image/ComputerNetwork/4.jpg" alt="4" style="zoom:100%;" />
<img src="/image/ComputerNetwork/5.jpg" alt="5" style="zoom:100%;" />
<img src="/image/ComputerNetwork/6.jpg" alt="6" style="zoom:100%;" />
<img src="/image/ComputerNetwork/7.jpg" alt="7" style="zoom:100%;" />
<img src="/image/ComputerNetwork/8.jpg" alt="8" style="zoom:100%;" />
<img src="/image/ComputerNetwork/9.jpg" alt="9" style="zoom:100%;" />
<img src="/image/ComputerNetwork/10.jpg" alt="10" style="zoom:100%;" />
<img src="/image/ComputerNetwork/11.jpg" alt="11" style="zoom:100%;" />
<img src="/image/ComputerNetwork/12.jpg" alt="12" style="zoom:100%;" />