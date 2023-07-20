---
title: C++11：新特性
date: 2023-2-27 15:55:06
tags:
- [CS]
- [C/C++]
- [C++11]
categories: 
- [CS]
- [C++11]
mathjax: true
description: C++11新特性。
top_img: 
cover: /image/covers/cpp11.png
---

# auto & decltype

- auto：让编译器在编译器就推导出变量的类型，可以通过=右边的类型推导出变量的类型。
- decltype：相对于auto用于推导变量类型，而decltype则用于推导表达式类型，这里只用于编译器分析表达式的类型，表达式实际不会进行运算。

1. auto的限制：

   1. auto的使用必须马上初始化，否则无法推导出类型
   2. auto在一行定义多个变量时，各个变量的推导不能产生二义性，否则编译失败
   3. auto不能用作函数参数
   4. 在类中auto不能用作非静态成员变量
   5. auto不能定义数组，可以定义指针
   6. auto无法推导出模板参数

   ```c++
   int i = 10;
   auto a = i, &b = i, *c = &i; // a是int，b是i的引用，c是i的指针，auto就相当于int
   auto d = 0, f = 1.0; // error，0和1.0类型不同，对于编译器有二义性，没法推导
   auto e; // error，使用auto必须马上初始化，否则无法推导类型
   
   void func(auto value) {} // error，auto不能用作函数参数
   
   class A {
       auto a = 1; // error，在类中auto不能用作非静态成员变量
       static auto b = 1; // error，这里与auto无关，正常static int b = 1也不可以
       static const auto c = 1; // ok
   };
   
   void func2() {
       int a[10] = {0};
       auto b = a; // ok
       auto c[10] = a; // error，auto不能定义数组，可以定义指针
       vector<int> d;
       vector<auto> f = d; // error，auto无法推导出模板参数
   }
   ```

2. auto与const, volatile

   1. 在不声明为引用或指针时，auto会忽略等号右边的引用类型和cv限定
   2. 在声明为引用或者指针时，auto会保留等号右边的引用和cv属性

   ```c++
   int i = 0;
   auto *a = &i; // a是int*
   auto &b = i; // b是int&
   auto c = b; // c是int，忽略了引用
   
   const auto d = i; // d是const int
   auto e = d; // e是int
   
   const auto& f = e; // f是const int&
   auto &g = f; // g是const int&
   ```

3. decltype: decltype则用于推导表达式类型，这里只用于编译器分析表达式的类型，表达式实际不会进行运算

   若exp是左值，decltype(exp)是exp类型的左值引用。

   如: `decltype(a += b) d = c`中的d即为int&, 因为`a += b`是一个左值。

4. auto和decltype的配合使用

   ```c++
   template<typename T, typename U>
   return_value add(T t, U u) {	// t和v类型不确定，无法推导出return_value类型
       return t + u;
   }
   
   template<typename T, typename U>
   decltype(t + u) add(T t, U u) { // t和u尚未定义
       return t + u;
   }
   
   // 返回类型后置的配合使用方法，为了解决函数返回值类型依赖于参数却难以确定返回值类型的问题
   template<typename T, typename U>
   auto add(T t, U u) -> decltype(t + u) {
       return t + u;
   }
   ```

# 左值引用、右值引用、移动语义、完美转发

[站内推荐: 引用折叠和完美转发](https://aflyingsheep.github.io/2023/02/27/cpp/forward/)

[左值引用、右值引用、移动语义、完美转发，你知道的不知道的都在这里 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/137662465)

返回值优化!!!!!!这里还没有看

[Return Value Optimization | Shahar Mike's Web Spot](https://shaharmike.com/cpp/rvo/)

[c++ - What are copy elision and return value optimization? - Stack Overflow](https://stackoverflow.com/questions/12953127/what-are-copy-elision-and-return-value-optimization)

```c++
// 右值引用
int a = 4;
int &&b = a;	// error! a是左值
int &&c = std::move(a);	// ok
```

1. 移动语义：转移所有权，对于别人的一块资源转为自己拥有，别人不再拥有也不再使用。

   ```c++
   // 移动方法：移动构造函数
   class A {
   public:
       A(A&& a) {
           this->data_ = a.data_;
           a.data_ = nullptr;
           std::cout << "move" << std::endl;
       }
   
       int *data_;
       int size_;
   }
   
   // main
   A a(10);
   A b = a;	// 浅拷贝
   A c = std::move(a);	// 调用移动构造函数
   
   // C++所有的STL都实现了移动语义；
   // 移动语义仅针对于那些实现了移动构造函数的类的对象，对于那种基本类型int、float等没有任何优化作用，还是会拷贝，因为它们实现没有对应的移动构造函数。
   ```

2. 完美转发：可以写一个接受任意实参的函数模板，并转发到其它函数，目标函数会收到与转发函数完全相同的实参，转发函数实参是左值那目标函数实参也是左值，转发函数实参是右值那目标函数实参也是右值。

# 列表初始化

在C++11中可以直接在变量名后面加上初始化列表来进行对象的初始化。

## std::initializer_list

使用STL过程中可能发现它的初始化列表可以是任意长度,使用`std::initializer_list`实现：

```c++
struct CustomVec {
    std::vector<int> data;
    CustomVec(std::initializer_list<int> list) {
        for (auto iter = list.begin(); iter != list.end(); ++iter) {
            data.push_back(*iter);
        }
    }
};
```

注意：std::initializer_list，它可以接收任意长度的初始化列表，但是里面必须是相同类型T，或者都可以转换为T。

# 并发(待补充)

。。。

# 智能指针

[站内推荐：智能指针](https://aflyingsheep.github.io/2023/02/28/cpp/ptr/)

# 基于范围的for循环：

```c++
vector<int> vec;

for (auto iter = vec.begin(); iter != vec.end(); iter++) { // before c++11
    cout << *iter << endl;
}

for (int i : vec) { // c++11基于范围的for循环
    cout << "i" << endl;
}
```

# 委托构造函数

委托构造函数允许在同一个类中一个构造函数调用另外一个构造函数，可以在变量初始化时简化操作。

注意委托环！！！

```c++
struct A {
    A(){}
    A(int a) { a_ = a; }

    A(int a, int b) : A(a) { b_ = b; }

    A(int a, int b, int c) : A(a, b) { c_ = c; }

    int a_;
    int b_;
    int c_;
};
```

# 继承构造函数

有时基类TestA可能有好多个构造函数。如果TestA有大量的构造函数，TestB只有一些成员函数，对于派生类而言，其构造等同于构造基类，这个时候我们就要写很多透传的构造函数。

```c++
class TestA {
public:
    TestA(string i): a1(i) {}
    TestA(int i) : a2(i) {}
    TestA(double i) : a3(i) {}
    ~TestA() {}
    string geta1() {
        return a1;
    }
    virtual string getb1() = 0;
private:
    string a1;
    int a2;
    double a3;
};

class TestB: public TestA{
public:
    TestB(string i):TestA(i),b1(i) {}
    TestB(int i) :TestA(i), b2(i) {}
    TestB(double i) :TestA(i), b3(i) {}
    ~TestB() {}
    virtual string getb1() override {
        return b1;
    }
    virtual void testb();
private:
    string b1;
    int b2;
    double b3;
};
```

c++11里提供一个规则，派生类可以通过使用using声明来声明继承基类的构造函数。这样的话就可以如下代码来代替上面的代码：

```c++
class TestA {
public:
    TestA(string i): a1(i) {}
    TestA(int i) : a2(i) {}
    TestA(double i) : a3(i) {}
    ~TestA() {}
    string geta1() {
        return a1;
    }
    virtual string getb1() = 0;
private:
    string a1;
    int a2;
    double a3;
};

class TestB: public TestA{
public:
    // 继承构造函数
	using TestA::TestA; 
	// ....
    virtual void testb();
private:
    string b1;
    int b2;
    double b3;
};
```

但是继承构造函数 只会初始化基类中的成员变量，对于派生类的成员变量，不会操作。可以通过使用初始化表达式来解决这个问题，例如：

```c++
class TestB: public TestA{
public:
    // 继承构造函数
	using TestA::TestA; 
	// ....
    virtual void testb();
private:
    string b1{"b1"};
    int b2{0};
    ...
};
```

这里另外有一点需要注意的事，如果基类构造函数参数有默认值，默认值会导致基类产生多个构造函数的版本。这样都会被派生类继承。所以在使用有参数默认值的构造函数的基类，必须小心。

另外私有的构造函数，不会被继承构造。

# nullptr

nullptr是c++11用来表示空指针新引入的常量值，在c++中如果表示空指针语义时建议使用nullptr而不要使用NULL，因为NULL本质上是个int型的0，其实不是个指针。举例：

```c++
void func(void *ptr) {
    cout << "func ptr" << endl;
}

void func(int i) {
    cout << "func i" << endl;
}

int main() {
    func(NULL); // 编译失败，会产生二义性
    func(nullptr); // 输出func ptr
    return 0;
}
```

# final & override

c++11关于继承新增了两个关键字，**final用于修饰一个类，表示禁止该类进一步派生和虚函数的进一步重载**，override用于修饰派生类中的成员函数，**标明该函数重写了基类函数**，如果一个函数声明了override但父类却没有这个虚函数，编译报错，使用override关键字可以避免开发者在重写基类函数时无意产生的错误。

```c++
struct Base {
    virtual void func() {
        cout << "base" << endl;
    }
};

struct Derived : public Base{
    void func() override { // 确保func被重写
        cout << "derived" << endl;
    }

    void fu() override { // error，基类没有fu()，不可以被重写
        
    }
};
```

```c++
struct Base final {
    virtual void func() {
        cout << "base" << endl;
    }
};

struct Derived : public Base{ // 编译失败，final修饰的类不可以被继承
    void func() override {
        cout << "derived" << endl;
    }

};
```

# default

c++11引入default特性，多数时候用于声明构造函数为默认构造函数，如果类中有了自定义的构造函数，编译器就不会隐式生成默认构造函数，如下代码：

```c++
struct A {
    int a;
    A(int i) { a = i; }
};

int main() {
    A a; // 编译出错
    return 0;
}
```

上面代码编译出错，因为没有匹配的构造函数，因为编译器没有生成默认构造函数，而通过default，程序员只需在函数声明后加上“`=default;`”，就可将该函数声明为 defaulted 函数，编译器将为显式声明的 defaulted 函数自动生成函数体，如下：

```c++
struct A {
    A() = default;
    int a;
    A(int i) { a = i; }
};

int main() {
    A a;
    return 0;
}
```

# delete

c++中，如果开发人员没有定义特殊成员函数，那么编译器在需要特殊成员函数时候会隐式自动生成一个默认的特殊成员函数，例如拷贝构造函数或者拷贝赋值操作符

我们有时候想禁止对象的拷贝与赋值，可以使用delete修饰，如下：

```c++
struct A {
    A() = default;
    A(const A&) = delete;
    A& operator=(const A&) = delete;
    int a;
    A(int i) { a = i; }
};

int main() {
    A a1;
    A a2 = a1;  // 错误，拷贝构造函数被禁用
    A a3;
    a3 = a1;  // 错误，拷贝赋值操作符被禁用
}
```

delele函数在c++11中很常用，std::unique_ptr就是通过delete修饰来禁止对象的拷贝的。

# explicit

explicit专用于修饰构造函数，表示只能显式构造，不可以被隐式转换。

```c++
struct A {
    A(int value) { // 没有explicit关键字
        cout << "value" << endl;
    }
};

int main() {
    A a = 1; // 可以隐式转换
    return 0;
}

/***********************************/

struct A {
    explicit A(int value) {
        cout << "value" << endl;
    }
};

int main() {
    A a = 1; // error，不可以隐式转换
    A aa(2); // ok
    return 0;
}
```

# constexpr

constexpr是c++11新引入的关键字，用于编译时的常量和常量函数，这里直接介绍constexpr和const的区别：

两者都代表可读，const只表示read only的语义，只保证了运行时不可以被修改，但它修饰的仍然有可能是个动态变量，constexpr修饰的才是真正的常量，它会在编译期间就会被计算出来，整个运行过程中都不可以被改变，constexpr可以用于修饰函数，这个函数的返回值会尽可能在编译期间被计算出来当作一个常量，但是如果编译期间此函数不能被计算出来，那它就会当作一个普通函数被处理。如下代码：

```c++
#include<iostream>
using namespace std;

constexpr int func(int i) {
    return i + 1;
}

int main() {
    int i = 2;
    func(i);// 普通函数
    func(2);// 编译期间就会被计算出来
}
```

- const用于修饰不能被修改的对象，但const对象的值通常在程序运行期间才能确定

- constexpr用于修饰常量表达式或可返回常量表达式的constexpr函数，在编译时能确定值。

- constexpr函数都是inline函数

# emun作用域

c++11新增有作用域的枚举类型，不带作用域的枚举类型可以自动转换成整形，且不同的枚举可以相互比较，代码中的红色居然可以和白色比较，这都是潜在的难以调试的bug，而这种完全可以通过有作用域的枚举来规避。

```c++
enum class AColor {
    kRed,
    kGreen,
    kBlue
};

enum class BColor {
    kWhite,
    kBlack,
    kYellow
};

int main() {
    if (AColor::kRed == BColor::kWhite) { // 编译失败
        cout << "red == white" << endl;
    }
    return 0;
}
```

同时带作用域的枚举类型可以选择底层类型，默认是int，可以改成char等别的类型。

```c++
enum class AColor : char {
    kRed,
    kGreen,
    kBlue
};
```

# 非受限联合体(待补充)

。。。

# sizeof

c++11中sizeof可以用的类的数据成员上：

```c++
struct A {
    int data[10];
    int a;
};

int main() {
    A a;
    cout << "size " << sizeof(a.data) << endl;	// before c++11
    cout << "size " << sizeof(A::data) << endl;	// c++11
    return 0;
}
```

想知道类中数据成员的大小在c++11中方便了许多，而不需要定义一个对象，在计算对象的成员大小。

# assertion

c++11引入static_assert声明，用于在**编译期间检查**，如果第一个参数值为false，则打印message，编译失败。

```c++
static_assert(true/false, message);
```

# 内存对齐(待补充)

。。。

# thread_local

c++11引入thread_local，用thread_local修饰的变量具有thread周期，每一个线程都拥有并只拥有一个该变量的独立实例，一般用于需要保证线程安全的函数中。

```c++
#include <iostream>
#include <thread>

class A {
   public:
    A() {}
    ~A() {}

    void test(const std::string &name) {
        thread_local int count = 0;
        ++count;
        std::cout << name << ": " << count << std::endl;
    }
};

void func(const std::string &name) {
    A a1;
    a1.test(name);
    a1.test(name);
    A a2;
    a2.test(name);
    a2.test(name);
}

int main() {
    std::thread(func, "thread1").join();
    std::thread(func, "thread2").join();
    return 0;
}

/*
	输出结果：
	thread1: 1
    thread1: 2
    thread1: 3
    thread1: 4
    thread2: 1
    thread2: 2
    thread2: 3
    thread2: 4
*/
```

验证上述说法，对于一个线程私有变量，一个线程拥有且只拥有一个该实例，类似于static。

# 基础数值类型

c++11新增了几种数据类型：long long、char16_t、char32_t等

# 正则表达式(待补充)

c++11引入了regex库更好的支持正则表达式

# 新增数据结构

- std::forward_list：单向链表，只可以前进，在特定场景下使用，相比于`std::list`节省了内存，提高了性能。

  ```c++
  std::forward_list<int> fl = {1, 2, 3, 4, 5};
  for (const auto &elem : fl) {
      cout << elem;
  }
  ```

  - 它实现为单链表，且实质上与其在 C 中实现相比无任何开销
  - 与 std::list 相比，此容器在不需要双向迭代时提供更有效地利用空间的存储

- std::unordered_set：基于hash表实现的set，内部不会排序，使用方法和set类似

- std::unordered_map：基于hash表实现的map，内部不会排序，使用方法和set类似

- std::array：数组，在越界访问时抛出异常，建议使用std::array替代普通的数组

- std::tuple: 元组类型

  [站内推荐：c++11: tuple](https://aflyingsheep.github.io/2023/02/28/cpp/tuple/)

# 新增算法(待补充)

。。。

# 随机数功能

c++11关于随机数功能则较之前丰富了很多，典型的可以选择概率分布类型，先看如下代码：

```c++
#include <time.h>

#include <iostream>
#include <random>

using namespace std;

int main() {
    std::default_random_engine random(time(nullptr));

    std::uniform_int_distribution<int> int_dis(0, 100); // 整数均匀分布
    std::uniform_real_distribution<float> real_dis(0.0, 1.0); // 浮点数均匀分布

    for (int i = 0; i < 10; ++i) {
        cout << int_dis(random) << ' ';
    }
    cout << endl;

    for (int i = 0; i < 10; ++i) {
        cout << real_dis(random) << ' ';
    }
    cout << endl;

    return 0;
}

/*
	输出：
	38 100 93 7 66 0 68 99 41 7
	0.232202 0.617716 0.959241 0.970859 0.230406 0.430682 0.477359 0.971858 0.0171148 0.64863
*/
```

代码中举例的是整数均匀分布和浮点数均匀分布，c++11提供的概率分布类型还有好多，例如伯努利分布、正态分布等，具体可以见最后的参考资料。

# Lambda表达式

