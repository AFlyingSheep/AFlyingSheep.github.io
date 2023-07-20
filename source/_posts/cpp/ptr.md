---
title: C++11：智能指针
date: 2023-2-28 12:13:06
tags:
- [CS]
- [C/C++]
- [C++11]
categories: 
- [CS]
- [C++11]
mathjax: true
description: C++11智能指针，包括shared_ptr, weak_ptr, unique_ptr。
top_img: 
cover: /image/shared_ptr/1.png 
---

## shared_ptr(有代码存疑)

shared_ptr使用了引用计数，每一个shared_ptr的拷贝都指向相同的内存，每次拷贝都会触发引用计数+1，每次生命周期结束析构的时候引用计数-1，在最后一个shared_ptr析构的时候，内存才会释放。

使用方法：

```c++
struct ClassWrapper {

    ClassWrapper() {
        cout << "construct" << endl;
        data = new int[10];
    }

    ~ClassWrapper() {
        cout << "deconstruct" << endl;
        if (data != nullptr) {
            delete[] data;
        }
    }

    void Print() {
        cout << "print" << endl;
    }

    int* data;
};

void Func(std::shared_ptr<ClassWrapper> ptr) {
    ptr->Print();
}

int main() {
    auto smart_ptr = std::make_shared<ClassWrapper>();
    auto ptr2 = smart_ptr; // 引用计数+1
    ptr2->Print();
    
    // 函数体内部引用计数+1，执行后-1，如果函数定义为引用传参则引用计数不变
    Func(smart_ptr); 
    smart_ptr->Print();
    ClassWrapper *p = smart_ptr.get(); // 可以通过get获取裸指针
    p->Print();
    return 0;
}
```

智能指针还可以自定义删除器，在引用计数为0的时候自动调用删除器来释放对象的内存，代码如下：

```c++
// 这块有点没看懂...
std::shared_ptr<int> ptr(new int, [](int *p){ delete p; });
```

通过shared_from_this()返回this指针，不要把this指针作为shared_ptr返回出来，因为this指针本质就是裸指针，通过this返回可能 会导致重复析构，不能把this指针交给智能指针管理。

```c++
class A {
      shared_ptr<A> GetSelf() {
        return shared_from_this();
        // return shared_ptr<A>(this); 错误，会导致double free
    }  
};
```

注意避免循环引用，可能导致内存永远不会释放，造成内存泄漏：

```c++
using namespace std;
struct A;
struct B;

struct A {
    std::shared_ptr<B> bptr;
    ~A() {
        cout << "A delete" << endl;
    }
};

struct B {
    std::shared_ptr<A> aptr;
    ~B() {
        cout << "B delete" << endl;
    }
};

int main() {
    auto aaptr = std::make_shared<A>();	// aaptr: 1
    auto bbptr = std::make_shared<B>();	// bbptr: 1
    aaptr->bptr = bbptr;	// bbptr: 2
    bbptr->aptr = aaptr;	// aaptr: 2
    return 0;
}
```

aptr和bptr的引用计数为2，离开作用域后aptr和bptr的引用计数-1，但是永远不会为0，导致指针永远不会析构，产生了内存泄漏，如何解决这种问题呢，答案是使用**weak_ptr**。

## weak_ptr

`weak_ptr`本身也是一个模板类，但是不能直接用它来定义一个智能指针的对象，只能配合`shared_ptr`来使用，可以将`shared_ptr`的对象赋值给`weak_ptr`，并且这样并不会改变引用计数的值。查看`weak_ptr`的代码时发现，它主要有`lock`、`swap`、`reset`、`expired`、`operator=`、`use_count`几个函数，与`shared_ptr`相比多了`lock`、`expired`函数，但是却少了`get`函数，甚至连`operator* `和 `operator->`都没有，可用的函数数量少的可怜，下面通过一些例子来了解一下`weak_ptr`的具体用法。

1. `weak_ptr`解决`shared_ptr`循环引用的问题
   定义两个类，每个类中又包含一个指向对方类型的智能指针作为成员变量，然后创建对象，设置完成后查看引用计数后退出，看一下测试结果：

   ```c++
   class CB;
   class CA
   {
   public:
       CA() { cout << "CA() called! " << endl; }
       ~CA() { cout << "~CA() called! " << endl; }
       void set_ptr(shared_ptr<CB>& ptr) { m_ptr_b = ptr; }
       void b_use_count() { cout << "b use count : " << m_ptr_b.use_count() << endl; }
       void show() { cout << "this is class CA!" << endl; }
   private:
       shared_ptr<CB> m_ptr_b;
   };
   
   class CB
   {
   public:
       CB() { cout << "CB() called! " << endl; }
       ~CB() { cout << "~CB() called! " << endl; }
       void set_ptr(shared_ptr<CA>& ptr) { m_ptr_a = ptr; }
       void a_use_count() { cout << "a use count : " << m_ptr_a.use_count() << endl; }
       void show() { cout << "this is class CB!" << endl; }
   private:
       shared_ptr<CA> m_ptr_a;
   };
   
   void test_refer_to_each_other()
   {
       shared_ptr<CA> ptr_a(new CA());
       shared_ptr<CB> ptr_b(new CB());
   
       cout << "a use count : " << ptr_a.use_count() << endl;
       cout << "b use count : " << ptr_b.use_count() << endl;
   
       ptr_a->set_ptr(ptr_b);
       ptr_b->set_ptr(ptr_a);
   
       cout << "a use count : " << ptr_a.use_count() << endl;
       cout << "b use count : " << ptr_b.use_count() << endl;
   }
   
   /*
   	测试结果：
   	CA() called!
       CB() called!
       a use count : 1
       b use count : 1
       a use count : 2
       b use count : 2
   
   */
   ```

   通过结果可以看到，最后CA和CB的对象并没有被析构，其中的引用效果如下图所示，起初定义完ptr_a和ptr_b时，只有①③两条引用，然后调用函数set_ptr后又增加了②④两条引用，当test_refer_to_each_other这个函数返回时，对象ptr_a和ptr_b被销毁，也就是①③两条引用会被断开，但是②④两条引用依然存在，每一个的引用计数都不为0，结果就导致其指向的内部对象无法析构，造成内存泄漏。

   ![1](/image/ptr/1.png)

   解决这种状况的办法就是将两个类中的一个成员变量改为`weak_ptr`对象，因为`weak_ptr`不会增加引用计数，使得引用形不成环，最后就可以正常的释放内部的对象，不会造成内存泄漏，比如将`CB`中的成员变量改为`weak_ptr`对象，代码如下：

   ```c++
   class CB
   {
   public:
       CB() { cout << "CB() called! " << endl; }
       ~CB() { cout << "~CB() called! " << endl; }
       void set_ptr(shared_ptr<CA>& ptr) { m_ptr_a = ptr; }
       void a_use_count() { cout << "a use count : " << m_ptr_a.use_count() << endl; }
       void show() { cout << "this is class CB!" << endl; }
   private:
       weak_ptr<CA> m_ptr_a;
   };
   
   /*
   	测试结果：
   	CA() called!
       CB() called!
       a use count : 1
       b use count : 1
       a use count : 1
       b use count : 2
       ~CA() called!
       ~CB() called!
   */
   ```

   ![2](/image/ptr/2.png)

2. weak_ptr常用函数

   `weak_ptr`中只有函数`lock`和`expired`两个函数比较重要，因为它本身不会增加引用计数，所以它指向的对象可能在它用的时候已经被释放了，所以在用之前需要使用`expired`函数来检测是否过期，然后使用`lock`函数来获取其对应的`shared_ptr`对象，然后进行后续操作：

   ```c++
   void test2()
   {
       shared_ptr<CA> ptr_a(new CA());     // 输出：CA() called!
       shared_ptr<CB> ptr_b(new CB());     // 输出：CB() called!
   
       cout << "ptr_a use count : " << ptr_a.use_count() << endl; // 输出：ptr_a use count : 1
       cout << "ptr_b use count : " << ptr_b.use_count() << endl; // 输出：ptr_b use count : 1
       
       weak_ptr<CA> wk_ptr_a = ptr_a;
       weak_ptr<CB> wk_ptr_b = ptr_b;
   
       if (!wk_ptr_a.expired())
       {
           wk_ptr_a.lock()->show();        // 输出：this is class CA!
       }
   
       if (!wk_ptr_b.expired())
       {
           wk_ptr_b.lock()->show();        // 输出：this is class CB!
       }
   
       // 编译错误
       // 编译必须作用于相同的指针类型之间
       // wk_ptr_a.swap(wk_ptr_b);         // 调用交换函数
   
       wk_ptr_b.reset();                   // 将wk_ptr_b的指向清空
       if (wk_ptr_b.expired())
       {
           cout << "wk_ptr_b is invalid" << endl;  // 输出：wk_ptr_b is invalid 说明改指针已经无效
       }
   
       wk_ptr_b = ptr_b;
       if (!wk_ptr_b.expired())
       {
           wk_ptr_b.lock()->show();        // 输出：this is class CB! 调用赋值操作后，wk_ptr_b恢复有效
       }
   
       // 编译错误
       // 编译必须作用于相同的指针类型之间
       // wk_ptr_b = wk_ptr_a;
   
   
       // 最后输出的引用计数还是1，说明之前使用weak_ptr类型赋值，不会影响引用计数
       cout << "ptr_a use count : " << ptr_a.use_count() << endl; // 输出：ptr_a use count : 1
       cout << "ptr_b use count : " << ptr_b.use_count() << endl; // 输出：ptr_b use count : 1
   }
   
   ```

3. 总结

   - weak_ptr虽然是一个模板类，但是不能用来直接定义指向原始指针的对象。
   - weak_ptr接受shared_ptr类型的变量赋值，但是反过来是行不通的，需要使用lock函数。
   - weak_ptr设计之初就是为了服务于shared_ptr的，所以不增加引用计数就是它的核心功能。
   - 由于不知道什么之后weak_ptr所指向的对象就会被析构掉，所以使用之前请先使用expired函数检测一下
     

## unique_ptr

`auto_ptr`本身是一个模板类，那么一般情况下直接用它来定义一个智能指针的对象。不过在当前不建议使用。

operator=也就是赋值运算符，是智能指针auto_ptr最具争议的一个方法，或者说一种特性，它的种种限制完全来自于这个赋值操作，作为面向的对象中的一部分，如果把一个对象赋值给另一个对象，那么两个对象就是完全一样的，但是这一点却在auto_ptr上打破了，智能指针auto_ptr的赋值，只是移交了所有权，将内部对象的控制所有权从等号的右侧转移到左侧，等号右侧的智能指针丧失对原有内部对象的控制，如果右侧的对象不检测内部对象的有效性，就会造成程序崩溃，测试如下：

```c++
void test5()
{
 auto_ptr<Example> ptr5(new Example(5));     // Example: 5(输出内容)
 auto_ptr<Example> ptr6 = ptr5;              // 没有输出

 if (ptr5.get())
     cout << "ptr5 is valid" << endl;        // 没有输出，说明ptr5已经无效，如果再调用就会崩溃

 if (ptr6.get())
     cout << "ptr6 is valid" << endl;        // ptr6 is valid(输出内容)

 ptr6->test_print();                         // in test print: number = 5(输出内容)
 //ptr5->test_print();                       // 直接崩溃 
}
```

测试auto_ptr作为参数，这是常常容易出错的情况，原因还是`operator=`的操作引起的，因为`auto_ptr`的赋值会转移控制权，所以你把`auto_ptr`的对象作为参数传递给一个函数的时候，后面再使用这个对象就会直接崩溃。

unique_ptr与auto_ptr是最像的，他设计之初就是为了替代auto_ptr，其实两者基本上没有区别，如果把auto_ptr限制一下，使其不能通过拷贝构造和赋值获得所有权，但是可以通过std::move()函数获得所有权，那基本上就变成了unique_pr，这一点通过下面的函数分析也可以看出，两者的函数基本一致。

`unique_ptr`作为一个模板类，可以直接用它来定义一个智能指针的对象，例如`std::unique_pr<Test> pa(new Test);`，查看`unique_ptr`的代码时发现，它主要有get、release、reset、operator*、operator->、operator=、swap、operator bool、get_deleter几个函数，相比于auto_ptr常用函数来说，只多了swap、operator bool、get_deleter这三个函数，基本上没什么变化。

## 总结

1. 对比auto_ptr和unique_ptr后发现，unique_ptr几乎只是将auto_ptr的operator=改为std::move()函数。
2. 现在标准库中只剩下了shared_ptr、weak_ptr和unique_ptr三个智能指针，weak_ptr是为了解决shared_ptr的循环引用问题而存在的，有其特定的使用情况，所以只剩下了shared_ptr和unique_ptr的选择，选择的标准就是看是否需要对原对象共享所有权，如果需要使用shared_ptr，如果不需要是独占所有权的使用unique_ptr。
3. unique_ptr并没有从根本上消除可能错误，仅仅是提高了犯错的成本，并且给出移动所有权的提示，但是在容器vector元素赋值时依然很隐晦，可能造成auto_ptr相同的错误。
   

