---
title: C++11：lambda表达式
date: 2023-2-28 20:06:06
tags:
- [CS]
- [C/C++]
- [C++11]
categories: 
- [CS]
- [C++11]
mathjax: true
description: C++11——lambda表达式，是 C++11 引入的一个“语法糖”。
top_img: 
cover: /image/covers/lambda.png 
---

# 介绍

lambda 来源于函数式编程的概念，也是现代编程语言的一个特点，，C# 3.5 和 Java 8 中就引入了 lambda 表达式。Lambda 表达式（Lambda Expression）是 C++11 引入的一个“语法糖”，可以方便快捷地创建一个“函数对象”。多用于在函数体内直接嵌套生成一个子函数，可以方便函数体内后续的调用。

从 C++11 开始，C++ 有三种方式可以创建/传递一个可以被调用的对象：

- 函数指针
- 仿函数(函数对象)
- Lambda表达式

## 函数指针

函数指针是从 C 语言继承下来的东西，比较原始，功能也比较弱：

1. 无法直接捕获当前的一些状态，所有外部状态只能通过参数传递（不考虑在函数内部使用 static 变量）。
2. 使用函数指针的调用无法 inline（编译期无法确定这个指针会被赋上什么值）。

```c++
// 一个指向有两个整型参数，返回值为整型参数的函数指针类型
int (*)(int, int);

// 通常我们用 typedef 来定义函数指针类型的别名方便使用
typedef int (*Plus)(int, int);

// 从 C++11 开始，更推荐使用 using 来定义别名
using Plus = int (*)(int, int);
```

## 仿函数

仿函数其实就是让一个类（class/struct）的对象的使用看上去像一个函数，具体实现就是在类中实现 operator()。

```c++
class Plus {
 public:
  int operator()(int a, int b) {
    return a + b;
  }   
};

Plus plus; 
std::cout << plus(11, 22) << std::endl;   // 输出 33
```

相比函数指针，仿函数对象可通过成员变量来捕获/传递一些状态。缺点就是，写起来很麻烦。

## Lambda表达式

Lambda 表达式在表达能力上和仿函数是等价的。编译器一般也是通过自动生成类似仿函数的代码来实现 Lambda 表达式的。上面的例子，用 Lambda 改写如下：

```c++
auto plus = [](int a, int b) -> int { return a + b };
```

# lambda 表达式的概念和基本用法

lambda 表达式定义了一个匿名函数，并且可以捕获一定范围内的变量。lambda 表达式的语法形式可简单归纳如下：

```c++
[ capture-list ] ( params ) opt -> ret { body; };
```

- opt: 函数声明选项，包括`mutable/exception/attribute`
- mutable：仅当 lambda 表达式是 mutable 时，才允许修改按值捕获的参数。
- exception：异常标识。
- attribute：属性标识。

lambda 表达式的捕获，其实就是将局部自动变量保存到 lambda 表达式内部。

**lambda 表达式不能捕获全局变量或 static 变量**。

举例而言，一个完整的 lambda 表达式看起来像这样：

```c++
auto f = [](int a) -> int { return a + 1; };
std::cout << f(1) << std::endl;  // 输出: 2
```

上面通过一行代码定义了一个小小的功能闭包，用来将输入加 1 并返回。

C++11 中允许省略 lambda 表达式的返回值定义，这样编译器就会根据 return 语句自动推导出返回值类型：

```c++
auto f = [](int a){ return a + 1; };
```

需要注意的是，初始化列表不能用于返回值的自动推导：

```c++
auto x1 = [](int i){ return i; };  // OK: return type is int
auto x2 = [](){ return { 1, 2 }; };  // error: 无法推导出返回值类型
```

这时我们需要显式给出具体的返回值类型。

另外，lambda 表达式在没有参数列表时，参数列表是可以省略的。因此像下面的写法都是正确的：

```c++
auto f1 = [](){ return 1; };
auto f2 = []{ return 1; };  // 省略空参数表
```

# 使用 lambda 表达式捕获列表

- [] 不捕获任何变量。
- [&] 捕获外部作用域中所有变量，并作为引用在函数体中使用（按引用捕获）。
- [=] 捕获外部作用域中所有变量，并作为副本在函数体中使用（按值捕获）。
- [=，&foo] 按值捕获外部作用域中所有变量，并按引用捕获 foo 变量。
- [bar] 按值捕获 bar 变量，同时不捕获其他变量。
- [this] 捕获当前类中的 this 指针，让 lambda 表达式拥有和当前类成员函数同样的访问权限。如果已经使用了 & 或者 =，就默认添加此选项。捕获 this 的目的是可以在 lamda 中使用当前类的成员函数和成员变量。this 指针只能按值捕获 [this] ，不能按引用捕获 [&this] 。

具体用法：

```c++
class A
{
    public:
    int i_ = 0;
    void func(int x, int y)
    {
        auto x1 = []{ return i_; };                    // error，没有捕获外部变量
        auto x2 = [=]{ return i_ + x + y; };           // OK，捕获所有外部变量
        auto x3 = [&]{ return i_ + x + y; };           // OK，捕获所有外部变量
        auto x4 = [this]{ return i_; };                // OK，捕获this指针
        auto x5 = [this]{ return i_ + x + y; };        // error，没有捕获x、y
        auto x6 = [this, x, y]{ return i_ + x + y; };  // OK，捕获this指针、x、y
        auto x7 = [this]{ return i_++; };              // OK，捕获this指针，并修改成员的值
    }
};
int a = 0, b = 1;
auto f1 = []{ return a; };               // error，没有捕获外部变量
auto f2 = [&]{ return a++; };            // OK，捕获所有外部变量，并对a执行自加运算
auto f3 = [=]{ return a; };              // OK，捕获所有外部变量，并返回a
auto f4 = [=]{ return a++; };            // error，a是以复制方式捕获的，无法修改
auto f5 = [a]{ return a + b; };          // error，没有捕获变量b
auto f6 = [a, &b]{ return a + (b++); };  // OK，捕获a和b的引用，并对b做自加运算
auto f7 = [=, &b]{ return a + (b++); };  // OK，捕获所有外部变量和b的引用，并对b做自加运算
```

需要注意的是，默认状态下 lambda 表达式无法修改通过复制方式捕获的外部变量。如果希望修改这些变量的话，我们需要使用引用方式进行捕获。

一个容易出错的细节是关于 lambda 表达式的延迟调用的：

```c++
int a = 0;
auto f = [=]{ return a; };      // 按值捕获外部变量
a += 1;                         // a被修改了
std::cout << f() << std::endl;  // 输出？
```

在这个例子中，lambda 表达式按值捕获了所有外部变量。在捕获的一瞬间，a 的值就已经被复制到f中了。之后 a 被修改，但此时 f 中存储的 a 仍然还是捕获时的值，因此，最终输出结果是 0。

如果希望 lambda 表达式在调用时能够即时访问外部变量，我们应当使用引用方式捕获。

从上面的例子中我们知道，按值捕获得到的外部变量值是在 lambda 表达式定义时的值。此时所有外部变量均被复制了一份存储在 lambda 表达式变量中。此时虽然修改 lambda 表达式中的这些外部变量并不会真正影响到外部，我们却仍然**无法修改它们**。

那么如果希望去**修改按值捕获的外部变量**应当怎么办呢？这时，需要显式指明 lambda 表达式为 mutable：

```c++
int a = 0;
auto f1 = [=]{ return a++; };             // error，修改按值捕获的外部变量
auto f2 = [=]() mutable { return a++; };  // OK，mutable
```

需要注意的一点是，被 mutable 修饰的 lambda 表达式就算没有参数也要写明参数列表。

# lambda表达式类型

lambda 表达式的类型在 C++11 中被称为“闭包类型（Closure Type）”。它是一个特殊的，匿名的非 nunion 的类类型。我们可以认为它是一个带有 operator() 的类，即仿函数。因此，我们可以使用 std::function 和 std::bind 来存储和操作 lambda 表达式：

```c++
std::function<int(int)> f1 = [](int a) { return a; };
std::function<int(void)> f2 = std::bind([](int a){ return a; }, 123);
```

另外，对于没有捕获任何变量的 lambda 表达式，还可以被转换成一个普通的函数指针：

```c++
using func_t = int(*)(int);
func_t f = [](int a){ return a; };
f(123);
```

lambda 表达式可以说是就地定义仿函数闭包的“语法糖”。它的捕获列表捕获住的任何外部变量，最终均会变为闭包类型的成员变量。而一个使用了成员变量的类的 operator()，如果能直接被转换为普通的函数指针，那么 lambda 表达式本身的 this 指针就丢失掉了。而没有捕获任何外部变量的 lambda 表达式则不存在这个问题。

这里也可以很自然地解释为何按值捕获无法修改捕获的外部变量。因为按照 C++ 标准，lambda 表达式的 operator() 默认是 const 的。一个 const 成员函数是无法修改成员变量的值的。而 mutable 的作用，就在于取消 operator() 的 const。

需要注意的是，没有捕获变量的 lambda 表达式可以直接转换为函数指针，而捕获变量的 lambda 表达式则不能转换为函数指针。看看下面的代码：

```c++
typedef void(*Ptr)(int*);
Ptr p = [](int* p){delete p;};  // 正确，没有状态的lambda（没有捕获）的lambda表达式可以直接转换为函数指针
Ptr p1 = [&](int* p){delete p;};  // 错误，有状态的lambda不能直接转换为函数指针
```

上面第二行代码能编译通过，而第三行代码不能编译通过，因为第三行的代码捕获了变量，不能直接转换为函数指针。