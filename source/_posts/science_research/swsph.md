---
title: SWSPH - 论文阅读笔记
date: 2022-11-15 22:28:06
tags:
- [Paper Reading]
categories: 
- [HPC]
mathjax: true
description: SWSPH-A Massively Parallel SPH Implementation for Hundred-Billion-Particle Simulation on New Sunway Supercomputer
top_img: /image/covers/picture/2.jpg
cover: /image/swsph/1.png 
---

## 优化方法

1. 域分解策略：

   1. 基于单元列表的成对交互：引入模拟域中分配均匀的空间网格，将整个粒子系统根据位空间坐标将粒子放置到cell中。

      进行粒子对交互时，每个中心单元根据自身和相邻的cell执行计算。第一层循环迭代当前cell，第二层迭代周围cell。

      确定粒子与周围cell的最小距离是否大于其支持域(剪枝)

      充分保持了数据的局部性，提高搜索成功率，减少分支数，有利于矢量化。

      <img src="C:\Users\赵阳\AppData\Roaming\Typora\typora-user-images\image-20221114160055315.png" alt="image-20221114160055315" style="zoom:80%;" />

2. 负载平衡——自适应粒子分割 & 体积自适应方案

   1. 自适应粒子分割

      - 传统SPH：为每个core分配统一数量的粒子，效率不高
      - 改进SPH：使用每个单元粒子最大支持域半径估算计算量，收敛区域使用树状自适应网格细化，直到计算量降至平均值以下，算法如下图：
      - 将单元半径设置为略大于支持域半径，减少无效搜索

      <img src="C:\Users\赵阳\AppData\Roaming\Typora\typora-user-images\image-20221114192107755.png" alt="image-20221114192107755" style="zoom:80%;" />

   2. 体积自适应方案

      1. 冲击模拟带来的问题：

         - 粒子压缩：局部负荷增加

         - 过度膨胀粒子：更大的支撑域收集足够粒子用于SPH插值，需要更大区域长度并增加没有膨胀波区域的计算负荷

      2. 体积自适应方案解决：

         - 粒子体积大于预设上限，母粒子拆分为8个子粒子(物理量继承母粒子物理数量)
         - 粒子体积小于预设下限，中心距离小于预设最大长度的一对粒子：合并为一个母粒子，物理量通过子粒子加权和计算

3. 通信优化策略

   将存储在每个节点的cell分为三种类型：**core cell, edge cell and ghost cell**

   1. 点对点通信模式：

      SPH中存在两种通信：粒子迁移(发送粒子到目标进程)、halo-exchange，采用非阻塞式点对点通信方法

      - 核心单元粒子计算与原子迁移、halo-exchange的堆叠

        <img src="C:\Users\赵阳\AppData\Roaming\Typora\typora-user-images\image-20221114215342162.png" alt="image-20221114215342162" style="zoom:80%;" />

   2. ghost单元冗余删除

      - 大多数框架，halo-exchange直接pack并发送，但包含大量冗余数据。
      - 优化：细化子单元，分析支持域有效半径，提前消除ghost单元中冗余粒子数据

4. 390-cores CPU细粒度任务映射策略

   1. 全共享模式：Sunway提供全共享模式：使用1个MPI进程+6个OpenMP线程能够控制6个CG，6*64个slave cores可以共享92GB内存

   2. 计算核心分组方案：

      - 根据牛顿第三定律(作用力与反作用力，计算量减半)：1. 找到比当前索引小的相邻cells并构建数据副本(之前已经计算过作用力，直接赋值即可); 2. 遍历比自己索引大的相邻cells，完成作用计算

      - 原分组方案：粗粒度并行(将整个i cell list分配给slave core，写入时j cell core的粒子数据也会被更新(牛顿第三定律))与细粒度并行(把每个cell list分配给64个slave core，但由于没有足够任务支持64个core parallel，将近一半的core处于忙等状态)

      - 现改进方案：将64个CPE按行划分为16个计算组(一组4个CPE)，每个计算组负责一个cell list，每个CPE负责计算一对cell

        <img src="C:\Users\赵阳\AppData\Roaming\Typora\typora-user-images\image-20221114231825086.png" alt="image-20221114231825086" style="zoom:80%;" />

        - 组中没有写入冲突，但组间写入冲突依旧存在——在列0的CPE的LDM中设置一个数据副本，当一个组所有成员完成cell list计算后，每个CPE将数据归约为第0列CPE的副本，并使用DMA更新内存数据
        - 数据归约：树约简方法更适合从核阵列的互联结构

   3. 数据布局优化与矢量化

      <img src="C:\Users\赵阳\AppData\Roaming\Typora\typora-user-images\image-20221114232342268.png" alt="image-20221114232342268" style="zoom:80%;" />

      1. 排序(根据相对位置，由CPE并行实现)
      2. AoS->AoSoA
      3. 可以使用simd加载指令从LDM读取8个连续的双数据并将它们放入向量寄存器中

