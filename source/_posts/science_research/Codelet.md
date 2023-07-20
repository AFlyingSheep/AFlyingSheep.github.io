---
title: Codelet调度模型与Swdarts
date: 2022-12-31 18:27:06
tags:
- [HPC]
categories: 
- [HPC]
mathjax: true
description: Codelet是一种针对E级计算机的需求而进行设计的细粒度并行、事件驱动的程序执行模型。
top_img: 
cover: /image/codelet/1.png
---

# Codelet程序执行模型

## 基本单元

- codelet程序执行模型的基本调度和执行单元
- 一个codelet是一些机器指令的集合（代码片段）
- codelet一旦开始执行，不会被中断或迁移
- codelet中不包括延迟时间长或等待的操作
- codelet间异步、并发，产生若干token(数据)作为输出

## 激发规则

1. 休眠状态：数据依赖未满足
2. 就绪状态：数据依赖满足
3. 激发状态：有空闲的计算单元且事件依赖被满足
4. 完成状态：完成计算

## Codelet Graph

计算任务被划分成大量的codelet, 这些codelet及其依赖关系构成一张数据流图，称位Codelet Graph(CDG)。CDG是一个有向无环图。

CDG被划分为一个个子图，每个子图被分配一个Threaded Procedure（TP），由TP对其调度和执行。

## Threaded Procedure

TP是异步的函数，以控制流的方式被调用，包括一个**上下文框架**和**CDG子图**。

- 上下文框架：为CDG中的codelet提供所需所有数据操作服务（初始化输入数据、为中间数据分配和释放空间、输出数据分配空间）
- TP被实例化后，绑定到一个核簇（cluster）上。TP的CDG子图中所有codelet被分配给该cluster调度执行。
- TP被实例化之前可以在cluster间迁移，以进行负载平衡。一旦被分配给一个cluster后，TP便不可移动。

## 抽象机器模型

抽象机器系统由若干计算节点（node）构成，计算节点以互联网络连接。每个计算节点由一个或者多个many-core chip构成，节点内chip以高速开关或总线互联。

Chip上的cluster以片上网络互联，每个cluster上有多个core。core分为计算单元（CU）和调度单元（SU）。CU负责执行codelet，SU负责：(1) 管理cluster所有硬件资源 (2) 在cluster间调度TP (3) 将处于就绪态的codelet依据一定调度策略给合适的CU执行。

![1](/image/codelet/1.png)

# Swdarts

## 抽象机器模型

为了利用申威众核异构平台，需要将Codelet抽象机器模型映射到国产异构众核平台上。

- 核组即可映射为Codelet抽象机器模型中的cluster，包含两种不同功能的core：
  - 主核：调度单元，负责任务分配和调度
  - 从核：计算单元，负责任务计算

![2](/image/codelet/2.png)

## Runtime system设计与实现

Runtime system主要负责任务的分配调度和任务间通信，是核心组件。

整个系统分为两层：

- 用户层：提供给用户描述、生成、调度可执行任务的接口，**用户通过所提供接口**对应用程序进行**数据流风格的抽象**，并使用接口将可执行任务交给下一层
- 运行时执行层：负责将以满足数据依赖关系的计算任务分配给空闲的硬件资源
  - 设置了任务队列，入队的任务表示依赖关系已经满足

### 前端接口设计

1. Task类：所有任务的抽象基类
   - 主要包括计算任务需要的上下文数据、可并行的计算任务、任务之间的数据依赖（CDG子图）
   - 对应Codelet中的TP
   - 可以发起生成一段可以并行执行的tasklet和task任务
   - 需要维护任务之间的链接关系来维护上下文数据的生命周期
     - execute(): task的执行函数，函数中可以做计算，也可以启动其他并行任务(task, tasklet)，再次声明任务之间的依赖关系
     - invoke(task) 启动并行执行的task任务
     - spawn_task(task_ptr) 将满足依赖关系的task任务放到任务队列中
2. Tasklet类：对应codelet，是运行时系统中的基本执行和调度单元
   - tasklet无需维护上下文数据，只需管理tasklet任务间的依赖关系
   - tasklet组成：依赖计数、执行函数、通知函数
     - spawn_tasklet(tasklet_ptr): 将tasklet放入tasklet队列中
     - release(): 释放tasklet的依赖计数，当计数为0时，调用spawn_tasklet()函数
     - execute(): 执行函数，执行过程调用激发函数，通过上述函数声明tasklet之间的依赖关系（？？）
     - can_spawn_on_cpe(): 表示tasklet在主核上还是从核上执行
3. enable_cpe_spawn类：与tasklet类基本相同，定义为可以在从核上运行的代码，只有执行函数
   - 创建该类原因：tasklet或enable_cpe_spawn依赖计数减为0时，被放入任务队列。依赖计数操作和通知操作会放在主核上执行，但是从核如果可以依赖计数操作和通知操作，需要对主核队列进行读写原子操作，代价大、成本高。
     - spawn__(): 主从核之间的任务传递，并在从核上执行计算任务
4. Runtime类：要负责任务的调度和执行，以及判断tasklet任务是否可以放到从核上执行，由用户声明。
   - make_tasklet(): 用于将函数定义或转换为tasklet
   - execute_and_wait(): 用于启动运行时系统和定义运行时系统停止的条件，并将数据流抽象化的应用交给运行时系统。
   - launch(Args…): 用户构造task时，通过launch()函数将任务需要的上下文传递给task

### 运行时系统设计

运行时系统：负责任务分配调度和执行；负责启动、管理从核

- 给从核队列中每个从核设置三种状态：初始化、空闲和繁忙
- 从核完成tasklet计算任务后，主核完成tasklet的通知任务，并将满足数据依赖的任务push到对应任务队列中
- 循环检查task任务队列，将队首任务取出并执行；循环检查tasklet任务队列，将队首tasklet根据用户声明交给主核或从核执行

运行时系统执行流程：

1. 用户完成应用的数据流化抽象，通过接口交给运行时系统
2. 将满足依赖关系的任务放入任务队列
3. 查询从核阵列状态，如果有空间资源，将任务分配计算
4. 从核计算完成，改变对应从核状态

![3](/image/codelet/3.png)

### 继承关系

- std::enable_shared_from_this\<task\>
  - task
    - tasklet
      - mem_fn_tasklet
    - task_context

