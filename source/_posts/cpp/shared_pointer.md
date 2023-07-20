---
title: C++智能指针：shared_ptr
date: 2022-12-12 21:22:06
tags:
- [CS]
- [C/C++]
- [C++11]
categories: 
- [CS]
- [C++11]
mathjax: true
description: C++引入智能指针，可用于动态资源管理，资源即对象的管理策略。
top_img: 
cover: /image/shared_ptr/1.png
---
使用raw pointer管理动态内存出现的问题：

1. 忘记delete内存，造成内存泄露；
2. 抛出异常时，无法执行delete，造成内存泄漏。

# 智能指针shared_ptrの原理

shared_ptr是最常用的C++11提供的智能指针。shared_ptr采用了引用计数器，多个shared_ptr中的T *ptr指向同一个内存区域（同一个对象），并**共同维护同一个引用计数器**。shared_ptr定义如下，记录同一个实例被引用的次数，当引用次数大于0时可用，等于0时释放内存。

从而可以在任何地方都不使用时自动删除相关指针，从而帮助彻底消除内存泄漏和悬空指针的问题。

每个 shared_ptr 对象在内部维护着两个内存位置：

1. **指向对象**的指针。
2. 用于**控制引用计数数据**的指针。

- **共享所有权如何在参考计数的帮助下工作的？**

  1. 当新的 shared_ptr 对象与指针关联时，则在其构造函数中，将与此指针关联的引用计数增加1。

  2. 当任何 shared_ptr 对象超出作用域时，则在其析构函数中，它将关联指针的引用计数减1。如果引用计数变为0，则表

     示没有其他 shared_ptr 对象与此内存关联，在这种情况下，它使用delete函数删除该内存。

- **注意避免循环引用**，shared_ptr的一个最大的陷阱是循环引用，循环，循环引用会导致堆内存无法正确释放，导致内存泄漏。循环引用在weak_ptr中介绍。

  ```c++
  temple<typename T>
  class SharedPtr {
  public:
     ...
  private:
      T *_ptr;
      int *_refCount;     //should be int*, rather than int
  };
  ```

一、 构造函数与析构函数

1. shared_ptr对象每次离开作用域时会自动调用析构函数，而析构函数并不像其他类的析构函数一样，而是在释放内存是先判断引用计数器是否为0。等于0才做delete操作，否则只对引用计数器左减一操作。

   ```c++
   ~SharedPtr() {
       if (_ptr && --*_refCount == 0) {
           delete _ptr;
           delete _refCount;
       }
   }
   ```

2. 构造函数：**默认构造函数的引用计数器为0，ptr指向NULL**

   ```c++
   SharedPtr() : _ptr((T *)0), _refCount(0) {
   
   }
   ```

3. 用**普通指针初始化智能指针时，引用计数器初始化为1**：

   创建空的 shared_ptr 对象

   ```c++
   explicit SharedPtr(T *obj) : _ptr(obj), _refCount(new int(1)) {
   }
   // 这里无法防止循环引用，若我们用同一个普通指针去初始化两个shared_ptr，
   // 此时两个ptr均指向同一片内存区域，但是引用计数器均为1，使用时需要注意
   ```

   因为带有参数的 shared_ptr 构造函数是 explicit 类型的，所以不能像这样`std::shared_ptr<int> p1 = new int();`隐式调用它构造函数。创建新的shared_ptr对象的最佳方法是使用`std :: make_shared`

   ```c++
   std::shared_ptr<int> p1 = std::make_shared<int>();
   ```

   **std::make_shared** 一次性为`int`对象和用于引用计数的数据都分配了内存，而`new`操作符只是为`int`分配了内存。

4. **拷贝构造函数**需要注意，用一个shared_ptr对象去初始化另一个shared_ptr对象时，**引用计数器加一，并指向同一片内存区域**：

   ```c++
   SharedPtr(SharedPtr &other) : _ptr(other._ptr), _refCount(&(++*other._refCount)) {
   
   }
   ```

	5. 赋值运算符的重载

    当用一个shared_ptr<T> other去给另一个 shared_ptr<T> sp赋值时，发生了两件事情：

    - sp指针指向发生变化，不再指向之前的内存区域，所以赋值前原来的_refCount要自减

    - sp指针指向other.ptr，所以other的引用计数器_refCount要做++操作。

    ```c++
    SharedPtr &operator=(SharedPtr &other) {
        if(this==&other)
            return *this;
     
        ++*other._refCount;
        if (--*_refCount == 0) {
            delete _ptr;
            delete _refCount;
        }
     
        _ptr = other._ptr;
        _refCount = other._refCount;
        return *this;
    }
    ```

二、 自定义运算符

1. 定义解引用运算符，直接返回底层指针的引用：

   ```c++
   T &operator*() {
       if (_refCount == 0)
           return (T*)0;
    
       return *_ptr;
   }
   ```

2. 定义指针运算符->

   ```c++
   T *operator->() {
       if(_refCount == 0)
           return 0;
    
       return _ptr;
   }
   ```

# shared_ptr 使用の注意事项

1. **缺少`++，––, [] `运算符，仅提供 `-->, *, ==`运算符。**

2. **NULL检测**

   当我们创建 shared_ptr 对象而不分配任何值时，它就是空的；普通指针不分配空间的时候相当于一个野指针，指向垃圾空间，且无法判断指向的是否是有用数据。

   ```c++
   std::shared_ptr<Sample> ptr3;
   if(!ptr3)
   	std::cout<<"Yes, ptr3 is empty" << std::endl;
   if(ptr3 == NULL)
   	std::cout<<"ptr3 is empty" << std::endl;
   if(ptr3 == nullptr)
   	std::cout<<"ptr3 is empty" << std::endl;
   
   /*
   输出结果：
   Yes, ptr3 is empty
   ptr3 is empty
   ptr3 is empty
   */
   ```

3. **创建 shared_ptr 的注意事项**

   **不要使用同一个原始指针构造 shared_ptr**

   创建多个 shared_ptr 的正常方法是使用一个已存在的shared_ptr 进行创建，而不是使用同一个原始指针进行创建。

   ```c++
   int *num = new int(23);
   std::shared_ptr<int> p1(num);
   
   std::shared_ptr<int> p2(p1);  // 正确使用方法
   std::shared_ptr<int> p3(num); // 不推荐
   
   std::cout << "p1 Reference = " << p1.use_count() << std::endl; // 输出 2
   std::cout << "p2 Reference = " << p2.use_count() << std::endl; // 输出 2
   std::cout << "p3 Reference = " << p3.use_count() << std::endl; // 输出 1
   ```

   假如使用原始指针`num`创建了p1，又同样方法创建了p3，当p1超出作用域时会调用`delete`释放`num`内存，此时num成了悬空指针，当p3超出作用域再次`delete`的时候就可能会出错。

4. **不要用栈中的指针构造 shared_ptr 对象**

   shared_ptr 默认的构造函数中使用的是`delete`来删除关联的指针，所以构造的时候也必须使用`new`出来的堆空间的指针。

   ```c++
   #include<iostream>
   #include<memory>
    
   int main()
   {
      int x = 12;
      std::shared_ptr<int> ptr(&x);
      return 0;
   }
   ```

   当 shared_ptr 对象超出作用域调用析构函数`delete` 指针`&x`时会出错。

5. **建议使用 make_shared**

   为了避免以上两种情形，建议使用`make_shared()<>`创建 shared_ptr 对象，而不是使用默认构造函数创建。

   ```c++
   std::shared_ptr<int> ptr_1 = make_shared<int>();
   std::shared_ptr<int> ptr_2 (ptr_1);
   ```

   另外不建议使用`get()`函数获取 shared_ptr 关联的原始指针，因为如果在 shared_ptr 析构之前手动调用了`delete`函数，同样会导致类似的错误。
