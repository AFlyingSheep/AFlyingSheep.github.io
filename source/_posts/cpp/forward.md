---
title: C++11：引用折叠和完美转发(巨NB)
date: 2023-2-27 22:55:06
tags:
- [CS]
- [C/C++]
- [C++11]
categories: 
- [CS]
- [C++11]
mathjax: true
description: 引用折叠、万能引用和完美转发，摘录自知乎ReFantasy。
top_img: 
cover: /image/covers/forward.png
---

# 引用折叠

**引用**的意思众所周知，当我们使用某个对象的别名的时候就好像直接使用了该对象，这也就是引用的含义。在C++11中，新加入了右值的概念。所以引用的类型就有两种形式：左值引用`T&`和右值引用`T&&`。

所谓的折叠，就是多个的意思。上面介绍引用分为左值引用和右值引用两种，那么将这两种类型进行排列组合，就有四种情况：

```c++
- 左值-左值 T& &
- 左值-右值 T& &&
- 右值-左值 T&& &
- 右值-右值 T&& &&
```

下面我们介绍引用折叠在模板中的应用：*完美转发*。在介绍完美转发之前，我们先介绍一下*万能引用*。

# 万能引用

**万能引用**并不是C++的语法特性，而是我们利用现有的C++语法，自己实现的一个功能。因为这个功能既能接受左值类型的参数，也能接受右值类型的参数。所以叫做万能引用。

```cpp
template<typename T>
ReturnType Function(T&& parem)
{
    // 函数功能实现
}
```

接下来，我们看一下为什么上面这个函数能**万能引用**不同类型的参数。

```c++
#include <iostream>
#include <boost/type_index.hpp>

using namespace std;
using boost::typeindex::type_id_with_cvr;  

template<typename T>
void PrintType(T&& param)
{
    // 利用Boost库打印模板推导出来的 T 类型
	cout << "T type：" << type_id_with_cvr<T>().pretty_name() << endl; 
    
    // 利用Boost库打印形参的类型
	cout << "param type:" << type_id_with_cvr<decltype(param)>().pretty_name() << endl;
}

int main(int argc, char *argv[])
{
	int a = 0;                              // 左值
	PrintType(a);                           // 传入左值

	int &lvalue_refence_a = a;              // 左值引用
	PrintType(lvalue_refence_a);            // 传入左值引用

	PrintType(int(2));                      // 传入右值
}
```

boost库用于看到模板内参数类型。

通过上面的代码可以清楚的看到，`void PrintType(T&& param)`可以接受任何类型的参数。下面，我们来仔细观察并分析一下`main`函数中对`PrintType()`的各个调用结果。

1. 传入左值

```c++
int a = 0;                              // 左值
PrintType(a);                           // 传入左值
/***************************************************/
输出：T type      : int &
      param type  : int &
```

我们将T的推导类型`int&`带入模板，得到实例化的类型：

```c++
void PrintType(int& && param)
{
   // ...
}
```

注：所有的引用折叠最终都代表一个引用，要么是左值引用，要么是右值引用。规则就是：如果任一引用为左值引用，则结果为左值引用。否则（即两个都是右值引用），结果为右值引用。

**编译器将T推导为 int& 类型。当我们用 int& 替换掉 T 后，得到 int & &&。也就是说，`int& &&`等价于`int &`。`void PrintType(int& && param)` == `void PrintType(int& param)`**

所以传入右值之后，函数模板推导的最终版本就是：

```c++
void PrintType(int& param)
{
   // ...
}
```

剩下的几个调用结果就很明显了：

```c++
int &lvalue_refence_a = a;              //左值引用
PrintType(lvalue_refence_a);            // 传入左值引用
/*
 * T type      : int &
 * T &&        : int & &&
 * param type  : int &
*/

PrintType(int(2));                      // 传入右值
/*
 * T type      : int
 * T &&        : int &&
 * param type  : int &&
*/
```

# 完美转发

有了万能引用。当我们既需要接收左值类型，又需要接收右值类型的时候，再也不用分开写两个重载函数了。那么，什么情况下，我们需要一个函数，既能接收左值，又能接收右值呢？

答案就是：转发的时候。

```c++
#include <iostream>
using namespace std;

// 万能引用，转发接收到的参数 param
template<typename T>
void PrintType(T&& param)
{
	f(param);  // 将参数param转发给函数 void f()
}

// 接收左值的函数 f()
template<typename T>
void f(T &)
{
	cout << "f(T &)" << endl;
}

// 接收右值的函数f()
template<typename T>
void f(T &&)
{
	cout << "f(T &&)" << endl;
}

int main(int argc, char *argv[])
{
	int a = 0;
	PrintType(a);//传入左值
	PrintType(int(0));//传入右值
}
```

我们执行上面的代码，按照预想，在main中我们给 PrintType 分别传入一个左值和一个右值。PrintType将参数转发给 f() 函数。f()有两个重载，分别接收左值和右值。

正常的情况下,`PrintType(a);`应该打印`f(T&)`,`PrintType(int());`应该打印`f(T&&)`。

**但是**，真实的输出结果是

```c++
f(T &);
f(T &);
```

**当外部传入参数给 PrintType 函数时，param既可以被初始化为左值引用，也可以被初始化为右值引用，取决于我们传递给 PrintType 函数的实参类型。但是，当我们在函数 PrintType 内部，将param传递给另一个函数的时候，此时，param是被当作左值进行传递的。** *应为这里的 param 是个具名的对象。我们不进行详细的探讨了。大家只需要己住，任何的函数内部，对形参的直接使用，都是按照左值进行的。*

**我们可以通过一些其它的手段改变这个情况，比如使用 std::forward 。**

使用万能引用的时候，如果传入的实参是个**右值(包括右值引用)**，那么，**模板类型 T 被推导为 实参的类型（没有引用属性）**，如果传入实参是个左值，T被推导为左值引用。**也就是说，模板中的 T 保存着传递进来的实参的信息，我们可以利用 T 的信息来强制类型转换我们的 param 使它和实参的类型一致。**

具体的做法就是，将模板函数`void PrintType(T&& param)`中对`f(param)`的调用，改为`f(std::forward<T>(param));`然后重新运行一下程序。输出如下：

```c++
f(T &);
f(T &&);
```

## 完美转发原理

`std::forward`是怎么利用到 T 的信息的呢？

```c++
/*
 *  精简了标准库的代码，在细节上可能不完全正确，但是足以让我们了解转发函数 forward 的了
 */ 

template<typename T>
T&& forward(T &param)
{
	return static_cast<T&&>(param);
}
```

**我们可以看到，不管T是值类型，还是左值引用，还是右值引用，T&经过引用折叠，都将是左值引用类型。也就是forward 以左值引用的形式接收参数 param, 然后 通过将param进行强制类型转换 static_cast<T&&> （），最终再以一个 T&&返回**

1. 传入 PrintType 实参是右值类型：

   根据以上的分析，可以知道T将被推导为值类型，也就是不带有引用属性，假设为 int 。那么，将T = int 带入forward。

   ```cpp
   int&& forward(int &param)
   {
   	return static_cast<int&&>(param);
   }
   ```

   `param`在forward内被强制类型转换为 int &&*(static_cast<int&&>(param))*, 然后按照int && 返回，两个右值引用最终还是右值引用。最终保持了实参的右值属性，转发正确。

2. 传入 PrintType 实参是左值类型：

   根据以上的分析，可以知道T将被推导为左值引用类型，假设为int&。那么，将T = int& 带入forward。

   ```c++
   int& && forward(int& &param)
   {
   	return static_cast<int& &&>(param);
   }
   ```

   引用折叠一下就是：

   ```cpp
   int& forward(int& param)
   {
   	return static_cast<int&>(param);
   }
   ```

   传递给 PrintType 左值，forward返回一个左值引用，保留了实参的左值属性，转发正确。