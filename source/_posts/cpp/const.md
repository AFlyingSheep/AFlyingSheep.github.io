---
title: C++：const在各个位置的含义
date: 2023-2-23 17:37:06
tags:
- [C++]
categories: 
- [C++]
mathjax: true
description: const常量，在函数各个位置有不同含义
top_img: 
cover: /image/covers/const.png
---

# const在函数的各个位置

1.`const`在函数返回值前，表明返回值为`const`

`const int getNum() { return i; }    // 表示返回值为const`

2.`const`在参数前面，表明参数为`const`，在函数体不能改变

`int getNum(const int num) {}   //num在函数体中不能被改变`

3.`const`在函数体后面，表示是类的常成员函数（函数体中可以把`this`看做是`const`，类的成员不能改变，只能调用其他常成员函数。**即不可以修改该类的成员变量**。）

`public int getNum() const;`

# const在指针定义的各个位置

`const int a = 2;`

`const int* p;

这种情况表示p是一个指向`const int`的指针。`p`可以改变，但是它指向的内容是不可变的。

`int const* p = &a;

这种情况表示p是一个指向int的const指针。p的地址不可以改变，但是它指向的内容可变。比如`*p = 3;`

`const int const *p = &a;`

`p`是一个指向`const int`的`const`指针。它指向的内容，和自身的地址都不可变。
