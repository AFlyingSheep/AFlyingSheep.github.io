---
title: Compare And Swap(CAS)原理分析
date: 2022-10-31 19:04:06
tags:
- [HPC]
categories: 
- [HPC]
- [CS]
mathjax: true
description: compare and swap，解决多线程并行情况下使用锁造成性能损耗的一种机制
top_img: 
cover: /image/CAS/1.png
---

# CAS概论

## CAS定义

**Compare and swap，解决多线程并行情况下使用锁造成性能损耗的一种机制**，CAS操作包含三个操作数——**内存位置**（V）、**预期原值**（A）和**新值**(B)。如果**内存位置的值与预期原值相匹配，那么处理器会自动将该位置值更新为新值。否则，处理器不做任何操作**。无论哪种情况，它都会在CAS指令之前返回该位置的值。CAS有效地说明了“我认为位置V应该包含值A；如果包含该值，则将B放到这个位置；否则，不要更改该位置，只告诉我这个位置现在的值即可。"

CAS操作是一条CPU的原子指令，所以不会有线程安全问题。

![1](/image/CAS/1.png)

## 加锁和CAS解决原子性问题的不同原理

考虑如下代码：

```java
//共享资源
static int i = 0;

public static void increase() {
    i++;
}

public static void main(String[] args) throws InterruptedException {
    Runnable r = () -> {
        for (int j = 0; j < 1000; j++) {
            increase();
        }
    };

    List<Thread> threads = new ArrayList<>();
    for (int j = 0; j < 10; j++) {
        Thread thread = new Thread(r);
        threads.add(thread);
        thread.start();
    }

    //确保前面10个线程都走完
    for (Thread thread : threads) {
        thread.join();
    }

    System.out.println(i);
}
```

对于i++，并不是原子性操作，导致10个线程执行后i的值并不是10*1000.

加锁解决的方式：

![2](/image/CAS/2.png)

CAS解决方式：

![3](/image/CAS/3.png)

# C++中的CAS操作

C++ 中的 CAS 操作用于操作原子变量，它是 ```atomic<T> ```的成员函数。

**在进行判等操作时，它执行的是物理上的比较，即直接比较内存值，而不是使用 `T` 的 `==` 操作符进行比较。此外，它允许虚假失败，也就是当前原子变量的内容与 `expected` 相等，但是它仍然返回 `false` ，但它不会修改 `expected` 。它需要放在循环中使用。**

[c++ CAS API接口(click me!)](https://blog.csdn.net/www_dong/article/details/119920236)

## 示例

```c++
#include <atomic>
 
template<class T>
struct node
{
    T data;
    node* next;
    node(const T& data) : data(data), next(nullptr) {}
};
 
template<class T>
class stack
{
    std::atomic<node<T>*> head;
 public:
    void push(const T& data)
    {
        node<T>* new_node = new node<T>(data);
 
        new_node->next = head.load(std::memory_order_relaxed);
 
        //std::memory_order_release: 本线程中，所有之前的写操作完成后才能执行本条原子操作
        //memory_order_relaxed: 不对执行的顺序作任何保证
        while(!std::atomic_compare_exchange_weak_explicit(
                                &head,
                                &new_node->next,
                                new_node,
                                std::memory_order_release,     
                                std::memory_order_relaxed))
                ; 
   }
};
 
int main()
{
    stack<int> s;
    s.push(1);
    s.push(2);
    s.push(3);
 
    return 0;
}
```

