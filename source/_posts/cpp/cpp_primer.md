---
title: C++ Primer
date: 2023-2-24 21:22:06
tags:
- [CS]
- [C/C++]
categories: 
- [CS]
mathjax: true
description: C++ Primer学习笔记，这个书牛牛牛。
top_img: /image/covers/picture/12.jpg
cover: /image/covers/cpp_primer.png
---

# 顺序容器

1. 容器的元素类型必须满足以下两个约束：元素类型必须支持赋值运算、元素类型的对象必须可以复制。

2. 不要存储end操作返回的迭代器。添加或删除deque或vector容器内的元素都会导致存储的迭代器失效。

3. vector提供了两个类成员函数: capacity和reserve。capacity用于获取容器需要分配更多的储存空间之前能够存储的元素总数；reserve操作告诉vector容器应该预留多少个元素的储存空间。

4. deque提供更复杂的数据结构，从队列两端插入和删除非常快，在中间操作代价更高。

   deque支持对所有元素的随机访问。

   deque在首尾插入元素不会使迭代器失效，在首位删除或在中间插入删除都会使迭代器失效。

5. 适配器：

   1. 默认的stack和queue基于deque容器实现，priority_queue在vector容器上实现。
   2. stack可以建立在vector, list, deque容器上；queue只能建立在list上，不能建立在vector上(要提供push_front运算)；priority_queue可以建立在vector, deque上，不能建立在list上(要提供随机访问)。
   3. 优先队列：允许用户为队列中存储的元素设置优先级

# 关联容器

1. 标准库定义了make_pair函数，由传递给它的两个实参生成一个新的pair对象

   ```c++
   pair<string, string> next_auth;
   string first, second;
   while (cin >> first >> second) {
       next_auth = make_pair(first, second);
       // process pair
   }
   ```

2. 关联容器的键不但有一个类型，还有一个比较函数，默认情况下为键类型定义的 < 操作符实现比较。

3. map.insert(e)返回pair<map<>::iterator, bool>, 如果键已在map中则关联值保持不变，返回的迭代器指向该pair并返回false；如果不在则插入新元素并返回迭代器和true；

   ```c++
   // insert重写单词统计
   map<string, int> word_count;
   string word;
   while (cin>> word) {
       pair<map<string, int>::iterator, bool> ret = word_count.insert(make_pair<word, 1>);
       if (!ret.second) {
           ++ret.first->second;
       }
   }
   ```

   set.insert(e)同样返回pair<set<>::iterator, bool>，与map类似，bool表示是否存在，迭代器指向插入或存在的值。

4. 在multimap与multiset中查找元素，可以在同一个键上调用`lower_bound, upper_bound`，分别返回该键关联的第一个元素与最后一个元素的下一位置。如果不存在，则`lower_bound == upper_bound`并且指向应该插入的位置。

   ```c++
   // 查找作者写的所有的书
   string search_item("Tom");
   
   authors_it begin = authors.lower_bound(search_item),
   			end = authors.upper_bound(search_item);
   while (begin != end) {
       std::cout << begin.second << std::endl;
       ++begin;
   }
   ```

   更直接的，可以直接调用equal_range函数取代以上两个函数，返回一对迭代器的pair对象。

   ```c++
   // 查找作者写的所有的书
   string search_item("Tom");
   
   pair<authors_it, authors_it> pos = authors.equal_range(search_item);
   while (pos.first != pos.second) {
       std::cout << pos.second << std::endl;
       ++pos.first;
   }
   ```


# 类

## 初始化表

1. 必须对任何const或引用类型成员以及没有默认构造函数的类类型的任何成员使用初始化式。
2. 成员被初始化的顺序是定义成员的次序，而不是初始化表的顺序。

## 隐式类型转换

1. 将构造函数声明为explicit，防止需要隐式转换的上下文中使用构造函数。

## static类成员

## 类类型对象

在 C++ 中，我们可以使用类名后加上一对括号来创建一个类的对象，同时可以使用类名和作用域解析运算符（`::`）来访问类的静态成员和静态函数。然而，当我们使用类名后加上一对括号时，编译器无法确定我们是要创建一个类的对象还是访问类的类型对象。因此，为了区分类对象和类类型对象，我们需要在类名后面添加关键字`class`。

例如，假设我们有以下的 C++ 代码，其中定义了一个名为`MyClass`的类和一个类类型对象`myClassObj`：

```c++
#include <iostream>

class MyClass {
public:
    int x;
    static int count;
    MyClass() {
        count++;
    }
};

int MyClass::count = 0;

int main() {
    MyClass obj1;
    MyClass::count = 10;

    // create a class object
    MyClass obj2;
    obj2.x = 42;

    // create a class type object
    class MyClass myClassObj;
    myClassObj.count = 5;

    std::cout << "obj1.count = " << obj1.count << std::endl;
    std::cout << "obj2.x = " << obj2.x << std::endl;
    std::cout << "myClassObj.count = " << myClassObj.count << std::endl;

    return 0;
}
```

在上面的示例中，我们在创建类类型对象`myClassObj`时，在类名`MyClass`前面加上了关键字`class`。这样，编译器就可以确定我们要创建一个表示`MyClass`类本身的对象。

使用关键字`class`声明类类型对象是一种好习惯，可以让代码更加清晰明了。另外，需要注意的是，关键字`class`只在初始化类类型对象时需要使用，在访问类的静态成员和静态函数时不需要使用。

# 复制控制

## 复制构造函数

1. 复制构造函数：只有单个形参，且形参是本类类型对象的引用。

2. C++支持两种初始化形式：直接初始化(将初始化式放在括号内，调用实参匹配的构造函数)、复制初始化(用"="符号，调用复制构造函数)。复制构造函数首先使用指定构造函数创建一个临时对象，然后用复制构造函数将临时对象复制到正在创建的对象。

   ```c++
   // 构造函数初始化举例
   #include <bits/stdc++.h>
   #include <string>
   
   using std::string;
   
   class Sales_item {
   	string x;
   public:
   	Sales_item(string input) : x(input) {
   		std::cout << "构造函数被调用" << std::endl;
   	}
   	Sales_item(const Sales_item& other) {
   		std::cout << "复制构造函数被调用" << std::endl;
   		x = other.x;
   	}
   }; 
   
   int main() {
   //	std::pair <int, double> p;
   //	p = std::make_pair(1.2, 1);
   //	std::cout << p.first << std::endl << p.second;
   	string s = "000";
   	// Sales_item si(s);
   	Sales_item si = string("000");
   	return 0;
   } 
   
   /*
   	输出结果：
   	构造函数被调用
   	复制构造函数被调用
   */
   ```

   注：如果构造函数是explicit，则`Sales_item si = string("000");`失效。

3. 复制构造函数是接受单个类类型引用形参(通常const)修饰的构造函数，一般不应设置为`explicit`。为了防止复制，类必须显式声明其复制构造函数为`private`。为防止友元和成员进行复制，可以声明一个`private`的复制构造函数但不定义，这样任何使用未定义的成员的任何尝试都会导致链接失败，编译时便会出错。

## 赋值操作符

1. 赋值操作符声明可以为：（右操作数一般作为const引用传递）

   ```c++
   class Sales_item{
   public:
       Sales_item& operator=(const Sales_item&);
   };
   ```

# 重载操作符

1. 操作符定义为非成员函数时，通常必须将它们设置为所操作类的友元，以访问类的私有部分。

## 重载操作符设计

1. 不要重置具有内置含义的操作符：重载逗号、取地址、逻辑与或等都不是好做法，这些操作符有内置含义。
2. 大多数操作符对类对象没有意义。可以考虑逻辑映射到操作符操作，如相等测试重载`==`，输入输出重载移位操作符，测试对象为空重载非操作符。
3. 复合赋值操作符。如重载`+`也要重载`+=`。
4. 相等和关系操作符。**将要用作关联容器键类型的类应该定义`<`和`==`操作符**。如果定义了相等操作符，也应该定义`!=`操作符。
5. 选择类函数或普通非成员函数的指导原则：
   1. 赋值、下标、调用、成员访问箭头(`=, [], (), ->`)等操作符必须定义为成员。
   2. 复合操作符通常定义为类的成员。
   3. 改变对象状态或与给定类型紧密联系的其他一些操作符，如自增，自减，解引用，一般定义为类的成员。
   4. 对称操作符最好定义为普通非成员函数，如算数操作符、相等操作符、关系操作符和位操作符。

## 输入输出操作符重载

```c++
// 重载输出操作符定义：
ostream&
    operator <<(ostream& os, const ClassType& object) {
    // prepare object
    
    // output
    os << ...
    return os;
}
// 如Sales_item
ostream&
    operator <<(ostream& os, const Sales_item& s) {
    os << s.isbn << std::endl;
    return os;
}
// 输入操作符
// 重要！！！输入操作符必须处理错误和文件结束的可能性
istream&
    operator >>(istream& in, Sales_item& s) {
 	double price;
    in >> s.isbn >> s.units_sold >> price;
    
    if (in) s.revenue = s.units_sold * price;
    // if input error, reset Sales_item
    else s = Sales_item();
    return in;
}
```

注：IO操作符必须为非成员函数，因为成员函数只能隐藏左操作数，所以预达到目标只能重载`ostream`，但是`ostream`却是在标准库中的，不能乱动...

## 下标操作符

类定义下标操作符时，一般需要定义两个版本：一个为非const成员并返回引用，另一个为const成员并返回const引用。

```c++
class Foo {
public:
    int &operator[] (const size_t);
    const int &operator [](const size_t) const;
private:
    vector<int> data;
}

int& operator [](const size_t index) {
    return data[index];
}
const int& operator [](const size_t index) const {
    return data[index];
}
```

## 成员访问操作符

指针支持的基本操作有解引用操作和箭头操作，我们的类可以定义：

```c++
class ScreenPtr {
public:
    // constructor and copy control memberes as before
    Screen &operator *() { return *ptr->sp };
    Screen *operator ->() { return ptr->sp };
    
    const Screen &operator *() const { return *ptr->sp };
    const Screen *operator ->() const { return ptr->sp };
    
private:
    ScrPtr *ptr;
}
```

## 重载箭头操作符

`->`表现为接受一个对象和一个成员名。由编译器处理获得成员的工作。所以，当我们编写如下代码：

`point -> action()`

等价于编写

`(point -> action)()`

换句话说，我们想调用对`point -> action()`求值的结果，编译器将代码进行如下求值：

1. 如果`point`为指针，指向具有名为`action`的成员的类对象，则将编译为调用该对象的`action`成员；
2. 否则，如果`action`是定义了`operator->`操作符的类的对象，则等价于`point.operator->()->action`。即执行`point`的`operator->()`，然后使用该结果再重复这三步。
3. 否则，代码错误。

**对重载箭头的返回值约束**：重载箭头必须返回指向类类型的指针，或者返回定义了自己的箭头操作符的类类型对象。

- 如果返回值是指针，则解引用，若没有该成员则编译器报错
- 如果返回值为类类型对象(或这种对象的引用)，则递归调用该操作符。

## 自增自减操作符重载

C++不要求自增或自减操作符一定作为类的成员，但由于操作符改变对象状态，更倾向于作为成员。

```c++
// 前缀操作符的实现(++ptr)：
class CheckedPtr {
public:
    CheckedPtr& operator++() {
        // 处理
        return *this;
    }
}

// 为了区别前后缀形式，后缀操作符函数接受一个额外的(无用的)int型形参，使用后编译器提供0作为形参的实参。
class CheckedPtr {
public:
    CheckedPtr operator++(int) {
        // 注：后缀返回的是旧值
        CheckedPtr ret(*this);
        ++*this;
        return ret;
    }
}

// 显式调用
CheckedPtr parr(ia, ia + size);
parr.operator++(0);	// 后缀形式
parr.operator++();  // 前缀形式
```

## 调用操作符和函数对象

函数对象：定义了调用操作符的类，其对象称为函数对象，即它们的行为类似函数的对象。

1. 函数对象用于标准库算法

   ```c++
   // old
   bool GT6(const string &s) {
       return s.size() >= 6;
   }
   
   vector<string>::sizetype wc = count_if(words.begin(), words.end(), GT6);
   
   // new
   class GT_cls {
   public:
       GT_cls(size_t val = 0): bound(val) {}
       bool operator() (const string &s) { return s.size() >= bound; }
   private:
       std::string::size_type bound;
   }
   
   vector<string>::sizetype wc = count_if(words.begin(), words.end(), GT_cls(6));
   ```

2. 函数对象的适配器和绑定器

   1. 绑定器，是一种函数适配器，通过将一个操作数绑定到给定值而将二元函数对象转换为一元函数对象。
   2. 求反器，是一种函数适配器，将为其函数对象的真值求反。

   标准库定义了两个绑定器的适配器: `bind1st, bind2nd`

   ```c++
   // 计算一个容器中所有小于等于10的元素个数
   count_if(vec.begin(), vec.end(), bind2nd(less_equal<int>(), 10));
   ```

   标准库定义了两个求反器: `not1, not2`

   ```c++
   // 对上个函数求反，即求所有大于10的元素个数
   count_if(vec.begin(), vec.end(), not1(bind2nd(less_equal<int>(), 10)));
   ```

## 其他运算符重载

1. 定义了`operator==`的类更容易与标准库一起使用，有些算法默认使用`==`运算符，如find。
2. 关联容器以及某些算法，默认使用<操作符。一般而言，关系操作符，诸如相等操作符应定义为非成员函数。
3. 无论形参为何种类型，赋值操作符必须定义为成员函数。且赋值操作符和复合赋值操作符应返回左操作数的引用。

# 转换与类类型

## 转换操作符

转换操作符是一种特殊的**类成员函数**，定义将类类型值转变为其他类型值的转换。**在类定义体内声明**，格式如下：

```c++
// 形式：operator type()
class SmallInt {
public:
    SmallInt(int i = 0): val(i) {
        if (i < 0 || i > 255)
            throw std::out_of_range("Bad!");
    }
    operator int() const { return val; }
private:
    std::size_t val;
}
```

其中，`type`表示**内置类型名、类类型名或由类型别名所定义的名字**。对任何可作为函数返回类型的类型(`void`除外)都可以定义转换函数。

1. 一般而言，不允许转换为数组或函数类型，但允许转换为指针类型(数据和函数的指针)以及引用类型。

2. 使用转换函数时，被转换的类型不必与所需要的类型完全匹配。例如：

   ```c++
   SmallInt si;
   double dval;
   si >= dval;	
   ```

   `SmallInt`首先转换为`int`类型，然后`int`转换为`double`的值。

3. **类类型转换以后不能在跟一个类类型转换！！**如果需要多个类类型转换，则代码会错误。

   ```c++
   // 假定另一个类Integral，可以转换为SmallInt但不能转换为Int
   Itergral intVal(1);
   SmallInt si(intval);	//ok, Integral->SmallInt
   int i = si;				//ok, SmallInt->int
   int j = intval			//error, Integral -x-> int
   ```

4. 标准转换可放在类类型转换之前。

## 实参匹配与转换

```c++
// 我们为SmallInt加上另外两个转换
class SmallInt {
public:
    SmallInt(int = 0);
    SmallInt(double);
    
    operator int() const { return val; }
    operator double() const {return val; }
    
    std::size_t val;
}
```

1. 实参匹配和多个转换操作符：

   一般而言，给出一个类与两个内置类型之间的转换是不好的做法，例如下述例子：

   ```c++
   void compute(int);
   void fp_compute(double);
   void extend_compute(long);
   
   SmallInt si;
   compute(si);	// SmallInt -> int
   fp_compute(si);	// SmallInt -> double
   extend_compute(si);	// error! ambiguous!
   ```

2. 实参匹配和构造函数转换

   ```c++
   void manip(const SmallInt &);
   
   double d; int i; long l;
   manip(d);	// ok: use SmallInt(double)
   manip(i);	// ok: use SmallInt(int)
   manip(l);	// error! ambiguous!
   ```

3. 当两个类定义了转换时的二义性

   ```c++
   class Integral;
   class SmallInt {
   public:
       SmallInt(Integral);	// convert from Integral to SmallInt
       // ...
   }
   class Integral {
   public:
       operator SmallInt() const; // convert from Integral to SmallInt
       // ...
   }
   
   // main
   void compute(SmallInt);
   Integral int_val;
   compute(int_val);	//error! ambigouos!
   
   compute(SmallInt(int_val));				// ok
   compute(int_val.operator SmallInt());	// ok
   ```

## 重载确定和类的实参

1. 在需要转换函数的实参时，编译器自动应用类的转换操作符或构造函数。于是函数重载确定由三部分组成：

   1. 确定候选函数集合：与被调用函数同名的函数。

   2. 确定可行函数：形参数目、类型与函数调用中的实参相匹配的候选函数。如果有转换操作，编译器还需确定使用哪个转换操作。

   3. 选择最佳匹配的函数。



2. 转换操作符之后的标准转换

   哪个函数是最佳匹配，可能依赖于匹配不同函数中是否涉及了一个或多个类类型转换：

   - 如果重载集中的两个函数可以使用同一转换函数匹配，则使用在转换之后或之前的标准转换序列的等级确定哪个函数为最佳匹配；

   - 否则，如果使用不同的转换操作，则认为两个转换是一样好的匹配，不管标准转换的等级如何。

3. 面对二义性转换，程序员可以使用强制转换显式指定应用哪个转换操作：

   ```c++
    void compute(int);
    void compute(double);
    
    SmallInt si;
    compute(static_cast<int>(si));
   ```

4. 标准转换和构造函数

   ```c++
   class SmallInt {
   public:
       SmallInt(int = 0);
   };
   
   class Integral {
   public:
       Integral(int = 0);
   };
   
   void manip(const Integral);
   void manip(const SmallInt);
   
   manip(10);			// error! ambiguous!
   
   // 即使一个类定义了实参需要标准转换的构造函数，该函数调用也具有二义性。
   // 原因见本节第二点
   
   // 使用显式构造消除二义性
   manip(SmallInt(10));
   manip(Integral(10));
   ```


# 面向对象编程

在C++中，通过基类的引用(或指针)调用虚函数时，发生动态绑定。引用(或指针)既可以指向基类对象也可以指向派生类对象，这是动态绑定的关键。用引用(或指针)调用的虚函数在**运行时确定**，被调用的函数是引用(或指针)所指对象的实际类型定义的。

## 定义基类和派生类

1. 派生类中虚函数的声明必须与基类中的定义方式完全匹配。但有一个例外：返回对基类型的引用(或指针)的虚函数，派生类中可以返回派生类的引用(或指针)。

2. 声明派生类不需要包含派生列表。

3. `virtual`与其他成员函数

   1. 要触发动态绑定，必须实现两个条件：

      - 只有指定为虚函数的成员函数才可以进行动态绑定。
      - 必须通过基类类型的引用或指针进行函数调用。

      当基类类型的引用和指针既可以指向基类类型，也可以指向派生类，因为派生类包含着基类。当使用指针或引用调用虚函数时，只有才运行时才可以确定指向的类型，并调用相应的函数。

      **引用和指针的静态类型与动态类型可以不同**，这是C++用以支持多态性的基石。

   2. 覆盖虚函数机制：派生类虚函数调用基类版本时，必须显式使用作用域操作符。如果派生函数忽略了这样做，则函数调用在运行时确定并且将是一个自身调用，从而导致无穷递归。

      ```c++
      // Bulk_item: Item_base
      Bulk_item derived;
      Item_base* baseP = &derived;
      
      // 调用基类版本的虚函数
      double b = baseP->Item_base::net_price(42);
      ```

   3. 虚函数与默认实参：派生类对于默认形参省略了该实参，则会使用基类的默认形参！！！

      所以基类与派生类的默认实参最好设置成一样的！

4. 公用、私有和受保护的继承

   - 公用继承：基类成员保持自己的访问级别。
   - 受保护的继承：基类成员的`public`成员为派生类的`protected`成员。
   - 私有继承：基类成员的所有成员在派生类中为`private`成员。

   1. 接口继承与实现继承：`private`和`protected`派生的类不继承基类的接口，这些派生通常被称为实现继承。

   2. 去除个别成员：尽管使用`private`或`protected`继承，但也可以使用`using`声明来从命名空间使用名字，保持访问等级：

      ```c++
      class Base {
      public:
      	int m;
      }; 
      
      class Extext: private Base {
      public:
      	using Base::m;
      };
      
      // main
      Base base;
      int m = base.m;	// ok
      ```

   3. 默认继承保护级别：`struct`保留字定义的类与用`class`定义的类唯一不同是默认的成员保护级别和默认的派生保护级别不同，其他无区别：

      ```c++
      class Base {/* ... */};
      struct D1 : Base {/* ... */};	// public继承
      class D2 : Base {/* ... */}		// private继承
      
      // 以下定义方式等价
      class D3 : public Base {/* ... */};
      struct D3 : Base {/* ... */};
      
      // 以下定义方式等价
      class D4 : Base {/* ... */};
      struct D4 : private Base {/* ... */};
      ```

5. 友元关系与继承：有缘关系不能继承。基类的友元对派生类的成员没有特殊访问权限。如果基类被授予友元关系，则只有基类具有特殊访问权限，该基类的派生类不能访问授予友元关系的类。

6. 继承与静态成员：如果基类定义了`static`成员，则整个继承层次中只有一个这样的成员，无论从基类派生出多少个派生类，每个`static`成员只有一个实例。

## 转换与继承

1. 引用转换不同于转换对象：

   - 可以将派生类型的对象传给希望接受基类引用的函数，引用直接绑定到该对象，但转换不会在任何方面改变派生类型对象，该对象仍是派生类型对象；
   - 将派生类型对象传给希望接受基类对象(而不是引用)的函数时，派生类对象的基类部分被复制到形参，形参类型便是固定的——编译与运行时均为基类对象。

2. 用派生类对象对基类对象进行初始化或赋值：

   - 基类一般(显式或隐式)定义自己的复制构造函数和赋值操作符，这些成员接受一个形参，**该形参是基类类型的`const`引用**。

     ```c++
     Item_base item;
     Bulk_item bulk;
     // ok, use Item_base::Item_base(const Item_base&)
     Item_base item(bulk);
     // ok, call Item_base::operator=(const Item_base&)
     item = bulk;
     
     /*
     	转换步骤：
     	1. Bulk_item对象转换为Item_base引用
     	2. 将该引用作为实参传给复制构造函数或赋值操作符
     	3. 使用Bulk_item的Item_base部分分别调用构造函数或赋值的Item_base对象的成员进行初始化和赋值
     	4. 执行完毕后，对象即为Item_base，包含Bulk_item的Item_base部分的副本，但实参的Bulk_item部分被忽略
     */
     ```

3. 派生类到基类转换的可访问性：

   1. 如果使用public继承，则用户代码和后代类都可以使用派生类到基类的转换；

   2. 如果使用private和protected继承，用户代码不能将派生类型对象转换为基类对象：

      - 如果是private继承，则从private继承类派生的类不能转换为基类；
      - 如果是protected继承，则后续派生类的成员可以转换为基类类型；

   3. 无论是什么派生访问标号，派生类本身都可以访问基类的public成员，因此派生类本身成员和友元总是可以访问派生类到基类的转换：

      ```c++
      class Base {
      private:
      	int n;
      public:
      	int m = 1;
      }; 
      
      class Extext: private Base {
      public:
      	using Base::m;
      	void function(const Base& base) {
      	std::cout << base.m;
      	} 
          // 这个函数体现了转换，本身的成员函数总是可以访问派生类到基类的转换的
      	void display() {
      		function(*this);
      	}
      };
      
      // main
      Extext ex;
      ex.display();
      ```

4. 基类到派生类的转换：没有从基类到派生类的自动转换，使用基类指针或引用实际绑定到派生类对象时，同样存在限制；如果知道基类到派生类转换是安全的，可以使用`static_case`或`dynamic_cast`进行转换。

## 构造函数与复制控制

1. 派生类可以在自己构造函数的初始化列表中向基类的构造函数进行参数传递。

2. 一个类只能初始化自己的**直接基类**！

3. 定义派生类复制构造函数

   ```c++
   class Base { /* ... */ };
   class Derived: public Base {
   public:
       // Base::Base(const Base&) 不会被自动调用，需要使用初始化函数Base(d)
       Derived(const Derived& d) {
           Base(d);
       }
   }
   ```

   初始化函数`Base(d)`将派生类对象d转换为它的基类部分的引用，并调用基类复制构造函数。

4. 派生类赋值操作符：如果派生类定义了自己的赋值操作符，该操作符必须对基类部分进行显式赋值：

   ```c++
   Derived &Derived::operator=(const Derived& rhs) {
       // 必须防止自身赋值
       if (this != &rhs) {
           Base::operator=(rhs);
           // do something...
       }
       return *this;
   }
   ```

5. 派生类的析构函数：每个析构函数只负责清除自己的成员。对象撤销顺序与构造函数相反，按继承层次依次向上调用。

6. 虚析构函数：删除指向动态分配对象的指针时，指针的静态类型可能与被删除对象的动态类型不同，可能会删除实际指向派生类对象的基类类型指针。

   要保证运行适当的析构函数，基类中的析构函数必须为虚函数。那么通过指针调用时，运行哪个析构函数将因指针所指对象类型的不同而不同。

   所以，即使析构函数没有工作要做，继承层次的根类也应该定义一个虚析构函数。

7. 构造函数和赋值操作符不是虚函数！

   - 构造函数在运行时，对象的动态类型是不完整的；
   - 虚函数要求形式完全相同，而赋值操作符中每个类都有一个与类本身相同的形参。

8. 构造函数和析构函数中的虚函数：

   在构造派生类对象时首先会运行基类的构造函数，而在撤销派生类对象时，会按照构造顺序的逆序撤销基类部分。在这两种情况下运行构造函数或析构函数，对象都是不完整的，编译器将对象的类型视为在构造和析构期间发生了变化。在基类构造函数或析构函数中，将派生类对象视为基类对象看待，**这对虚函数的绑定有影响**。

   **如果在构造函数或析构函数中调用虚函数，则运行的是为构造函数或析构函数自身类型定义的版本**。

## 纯虚函数

```c++
class Disc_item: public Item_base {
public:
    double net_price(std::size_t) const = 0;
}
```

在函数形参表后面写上=0以指定纯虚函数。

含有(或继承)一个或多个纯虚函数的类是抽象基类，除了作为抽象基类的派生类的对象的组成部分，不能创建抽象类型的对象。

## 容器与继承

因为派生类对象在赋值给基类对象时会被"切掉"，所以容器与通过继承相关的类型不能很好的融合。

## 句柄类与继承

1. 句柄类存储和管理基类指针。指针所指对象的类型可以变化，既可以指向基类类型对象又可以指向派生类型对象。用户通过句柄类访问继承层次的操作。句柄类类似指针执行操作，**虚成员**的行为将在运行时根据句柄实际绑定的对象类型而变化。

2. 复制未知类型：句柄类经常需要在不知道对象的确切类型时分配已知对象的新副本。解决这个问题的通常方法是定义**虚操作**进行复制，称该操作为clone。

   ```c++
   // 基类
   class Item_base {
   public:
       virtual Item_base* clone() const {
           return new Item_base(*this);
       }
   }
   
   // 派生类
   class Bulk_item {
   public:
       virtual Bulk_item* clone() const {
           return new Bulk_item(*this);
       }
   }
   ```

3. 句柄的使用

   1. 使用带比较器的关联容器

      ```c++
      inline bool compare(const Sales_item& lhs, const Sales_item& rhs) {
          return lhs->book() < rhs->book();
      }
      
      typedef bool (*Comp) (const Sales_item&, const Sales_item&);
      std::multiset<Sales_item, Comp> items(compare);
      ```

   2. 使用句柄执行虚函数

      ```c++
      double Basket::total() const {
          double sum = 0.0;
          for (const_iter iter = items.begin(); 
               iter != items.end(); 
               iter = items.upper_bound(*iter)) {
              sum += (*iter)->net_price(items.count(*iter));
          }
      }
      ```

      
