---
title: C#中调用Java的类与方法
date: 2022-01-22 15:50:05
categories: 
- Other tools
tags:
- Tools
description: 在本篇文章中，我们使用ikvm工具来实现C#中可以调用Java的类与方法。
top_img:
cover: /image/covers/cjava.png
---

C#中调用Java的类与方法

- 核心：ikvm生成jar的dll文件，在c#中进行引用。

- 易踩坑：java1.7 对应ikvm7；java1.8 对应ikvm8

- 步骤：

  1. 在Idea中，添加非默认文件夹下的包，如图中com。

  ![步骤1](/image/C_sharp_Java/1.png)

  2. 编写类文件 ChatClient和ChatServer。
  ![步骤2](/image/C_sharp_Java/2.png)
  
  3. 使用idea对工程进行build，建立jar包。

  4. 在jar包的目录下cmd输入：ikvmc -out:hello.dll Chat.jar。

  5. 将生成的dll文件和IKVM的几个文件添加到引用。
  ![步骤5](/image/C_sharp_Java/5.png)

  6. 在C#中引用包，using com;

  7. 在类中即可调用。
  ![步骤7](/image/C_sharp_Java/7.png)
