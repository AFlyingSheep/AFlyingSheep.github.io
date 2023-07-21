---
title: 神威混合编译
date: 2023-7-21 23:56:06
tags:
- [C++]
- [Sunway]
categories: 
- [C++]
- [Sunway]
mathjax: true
description: 神威支持混合编译，支持从核部分C++等新特性。非常好玩。
top_img: 
cover: /image/covers/sunway.png
---

近期师兄让我学习了神威的混合编译部分，发现这真是个神奇的东西，可以不用再分两个文件写从核代码，也可以灵活运用宏来分别编译主从核部分、自动推断函数类型，甚至可以把对象从主核传送到从核（这真的很方便）。

# 神威主从核编译概述

1. 编译器增加-mhybrid-coding选项，支持C++主从核代码在同文件内混合编写，默认为主核代码，从核入口函数增加attribute kernel属性。在编译时，分别使用-mslave和-mhost生成主从核对应的.o文件，最后将其链接起来。
2. 没有被显示调用的主从核函数要增加arribute slave和arribute hostslave属性，具体见文档。
3. 更多的细节见文档，本文主要就主从核crtp进行分享。

# 神威CRTP

本文中，首先编写了task基类，其中有`Initialize()`，`Finalize() `和 `Self()` 三个方法，基类中均是使用CRTP方式进行调用；接着编写了两个派生类，其`Self() `方法分别对每个数组的数值加一或减一。主要代码如下所示：

## task.h

```c++
#ifndef SWSPH_TASK_H_
#define SWSPH_TASK_H_
#include <vector>

namespace swsph {

template <typename T>
class Task {
 public:
  Task(std::vector<int>& arr) : arr_{arr} {};

  void Initialize() {
    static_cast<T*>(this)->Initialize();
  }

  void Finalize() {
    static_cast<T*>(this)->Finalize();
  }

  void Self(int index) {
    static_cast<T*>(this)->Self(index);
  }

  std::vector<int>& arr_;
};

};  // namespace swsph

#endif
```

## task_acc.h

```c++
#ifndef SWSPH_TASK_ACC_BF_H_
#define SWSPH_TASK_ACC_BF_H_

#include "task.h"

namespace swsph {

class CalcAccBf : public Task<CalcAccBf> {
 public:
  CalcAccBf(std::vector<int>& arr) : Task<CalcAccBf>(arr), ref_arr(arr) {}

  void Initialize();

  void Self(int index);

  void Finalize();

  std::vector<int>& get_buf() { return acc_buf_; }

 private:
  std::vector<int> acc_buf_;
  std::vector<int>& ref_arr;
};

class CalcAccMinus : public Task<CalcAccMinus> {
 public:
  CalcAccMinus(std::vector<int>& arr) : Task<CalcAccMinus>(arr), ref_arr(arr){}

  void Initialize();

  void Self(int index);

  void Finalize();

  std::vector<int>& get_buf() { return acc_buf_; }

 private:
  std::vector<int> acc_buf_;
  std::vector<int>& ref_arr;
};

}  // namespace swsph

#endif 
```

## task_acc.cc

```c++
#include "task_acc_bf.h"
#include <crts.h>
#include <utility>
#include <iostream>

namespace swsph {
    [[gnu::slave]] void CalcAccBf::Initialize() {
      printf("Slave %d, plus init.\n", _PEN);
    }

    [[gnu::slave]] void CalcAccBf::Finalize() {
      printf("Slave %d, plus final.\n", _PEN);
    }

    [[gnu::slave]] void CalcAccBf::Self(int index) {
      ref_arr[index]++;
    }

    [[gnu::slave]] void CalcAccMinus::Initialize() {
      printf("Slave %d, minus init.\n", _PEN);
    }

    [[gnu::slave]] void CalcAccMinus::Finalize() {
      printf("Slave %d, minus final.\n", _PEN);
    }

    [[gnu::slave]] void CalcAccMinus::Self(int index) {
      ref_arr[index]--;
    }
}
```

## main.cc

```c++
#include <crts.h>
#include <vector>
#include <iostream>
#include "task.h"
#include "task_acc_bf.h"
#include "main.h"

#define SW64_SPAWN(fn, args) athread_spawn_cgs((fn), (args))
#define SW64_JOIN() athread_join_cgs()

#define VECTOR_LENGTH 128

template <typename T> [[gnu::slave]] 
void TimeIntegration::for_each_compute(swsph::Task<T>& task) {
  int nr_cells = num_size_;
  int cell_start = _PEN * nr_cells / 64;
  int cell_end = (_PEN + 1) * nr_cells / 64;

  printf("Slave %d says: run! Start: %d, end: %d.\n", _PEN, cell_start, cell_end);
  task.Initialize();
  for (int index = cell_start; index != cell_end; index++) {
    task.Self(index);
  }
  task.Finalize();
}

[[gnu::kernel]] 
void TimeIntegration::calculate_plus_test() {
#ifdef __sw_slave__
    swsph::CalcAccBf cal(arr_);
    for_each_compute(cal);
#else
#endif
}

[[gnu::kernel]] 
void TimeIntegration::calculate_minus_test() {
#ifdef __sw_slave__
    swsph::CalcAccMinus cal(arr_);
    for_each_compute(cal);
#else
#endif
}

[[gnu::host]] 
void TimeIntegration::run() {
    set_num_size(VECTOR_LENGTH);
    arr_.resize(num_size_, 0);
    calculate_plus_test();
    athread_join();
    for (int i = 0; i < VECTOR_LENGTH; i++) {
      printf("%d", arr_[i]);
    }
    printf("\n");
    calculate_minus_test();
    athread_join();
    for (int i = 0; i < VECTOR_LENGTH; i++) {
      printf("%d", arr_[i]);
    }
    printf("\n");
}

int main() {
    athread_init();
    TimeIntegration tm;
    tm.run();
}
```

函数前面使用`[[gnu::kernel]]`修饰一下，编译器自动识别并分别编译。这样可以实现混合编译，不需要单独写一个从核.c文件。更重要的是支持C++，这个确实方便了很多。

## 编译选项

```makefile
fake_path/swg++ -mhybrid-coding task_acc_bf.cc -c -mslave -o build/task-slave.o
fake_path/swg++ -mhybrid-coding main.cc -c -o build/host.o
fake_path/swg++ -mhybrid-coding main.cc -c -mslave -o build/slave.o -msimd
fake_path/swg++ -mhybrid build/task.o build/host.o build/slave.o build/task-slave.o -o build/for-each
```

# 总结 & Reference

本次是对于混合编译的初体验，对于lambda表达式的传递、对象传递等很多内容还需要继续学习体验，后续再更。

Reference: 神威混合编译手册。
