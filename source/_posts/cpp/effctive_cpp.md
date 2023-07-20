---
title: effective-c++阅读笔记
date: 2023-5-23 22:06:06
tags:
- [CS]
- [C/C++]
categories: 
- [CS]
- [C/C++]
mathjax: true
description: 阅读effective c++笔记：55个改善程序的条款。
top_img: 
cover: /image/covers/effective-cpp.png
---


# 让自己习惯C++

## 条款1：视 C++ 为一个语言联邦

1. C++高效编程守则视状况而变化，取决于使用C++的哪个部分：
   1. C
   2. Object-Oriented C++: 包括class, 封装, 继承, 多态, 虚函数动态绑定等等。
   3. Template C++: 泛型编程，TMP(模板元编程)。
   4. STL

## 条款2：尽量以 const、enum、inline 替换 #define

1. 当使用`#define PI 3.14` 声明一个常量(`#define`并不属于程序的一部分)时，可能未被编译器看到。解决方法便是使用`const  double PI = 3.14.`存在两种特殊情况：

   1. 定义常量指针：常量定义式通常被放在头文件中，有必要将指针声明为`const`(而不是指针所指内容)，如：

      `const char* const authorName = "Kerbal;"`

      然而`string`通常比`char*-based`更合适：

      `const std::string authorName = "Kerbal";`

   2. `class`专属常量：将常量的作用域限制于class内，则必须让其成为类的成员，同时确保常量最多存在一份实体，则让其成为static成员：

      ```c++
      class GamePlayer{
      private:
      	static const int NumTerns = 5; // 常量声明式
      	int score[NumTerns];
      	...
      };
      ```

      只要不取地址则无需提供NumTerns的定义式。如果坚持取地址或编译器坚持看到一个定义式，则需要在**实现文件**中提供：`const int GamePlayer::NumTerns;`,并且定义时无需赋初值。

   3. 如果编译器不允许在声明时赋初值，则需要将初值放在定义式中。然而编译器又需要在编译期间确定数组的长度，则需要enum-hack：

      还有一点好处：enum-hack不会被取地址，不会导致非必要的内存分配。

      ```c++
      class GamePlayer{
      private:
      	enum{
      		NumTerns = 5 // 令 NumTerns 成为 5 的记号名称
      	};
      	int score[NumTerns];
      	//...
      };
      ```

2. 对于形似函数的宏，最好改用inline函数替换掉`#define`

## 条款3：尽可能使用const

1. `const`可以修饰很多东西。而对于指针的修饰，详见_____文章

2. 对于STL的迭代器，`const`同样能够进行修饰：

   ```c++
   const std::vector<int>::iterator iter = vec.begin();	// 指针不可以变化
   std::vector<int>::const_iterator cIter = vec.begin();	// 所指内容不变
   ```

3. 编译器强制bitwise constness，但编写程序应该使用概念上的常量性，可以使用`mutable`打破编译器的bitwise constness。

4. 当const和non-const成员函数都有着实质等价的实现时，可以复用：

   1. 先对non-const进行static_cast转换为const。
   2. 调用后使用const_cast去除const性。

   ```c++
   class TextBlock {
   public:
       const char& operator[] (std::size_t position) const { 
           ...
           return text[position];
       }
       char& operator[] (std::size_t position) {
           return const_cast<char&>(
           	static_cast<const TextBlock&>(*this)[position]
           );
       }
   };
   ```


## 条款4：确定对象使用前初始化

1. static对象分为local static(在函数内部的static对象)与non-local static对象(其他的static)，问题是如果某个编译单元内的某个non-local static对象依赖于其他编译单元的non-local static对象，然而这个对象可能尚未初始化。

   解决办法：将每个non-local static对象替换为local static对象(单例设计模式)：

   ```c++
   // In file 1
   // 服务器建立
   class FileSystem{
   public:
   	// ...
   	std::size_t numDisk()const; // 成员函数
   	// ....
   };
   extern FileSystem tfs; // 准备给客户使用的对象
   
   // In file 2
   class Directory {
     ...  
   };
   Directory::Directory(params) {
       ...
       // 出现问题：万一这个文件先编译，则会出现以下错误：无法解析的外部符号：class FileSystem tfs
       std::size_t disks = tfs.numDisk();
       ...
   }
   
   // 使用local static对象替换(单例设计模式)
   FileSystem& tfs() {
       static FileSystem fs;
       return fs;
   }
   
   Directory::Directory(params) {
       ...
       // 保证reference指向了已经初始化的对象
       std::size_t disks = tfs().numDisk();
       ...
   }
   ```

   静态数据成员初始化与一般数据成员初始化不同。初始化时可以不加 static，但必须要有数据类型。被 private、protected、public 修饰的 static 成员变量都可以用这种方式初始化。静态数据成员初始化的格式为：＜数据类型＞＜类名＞::＜静态数据成员名＞=＜值＞

   出现在类体外的函数定义不能指定关键字static；

   一般程序的由new产生的动态数据存放在堆区，函数内部的自动变量存放在栈区，静态数据（即使是函数内部的静态局部变量）存放在全局数据区。自动变量一般会随着函数的退出而释放空间，而全局数据区的数据并不会因为函数的退出而释放空间。

# 构造/析构/赋值运算

## 条款5：了解 C++ 默认编写调用的函数

1. 编译器产出的析构函数是个non-virtual，除非这个class的base class自身声明有virtual的析构函数。
2. 内含reference或内含const的成员若想实现赋值动作则必须自己定义copy assignment操作符。因为C++不允许引用改指不同的对象，更改const成员同样不合法。
3. 如果某个base classes将copy assignment操作符声明为private，则编译器拒绝为其derived classes生成copy assignment。

## 条款6：若不想使用编译器自动生成的函数，就该明确拒绝

书中根据条款5.3编写了禁止拷贝基类，不过有点多余，用下面代码其实就可以：

```c++
class DontCopyMe {
public:
	DontCopyMe();
    DontCopyMe(const DontCopyMe& dcm) = delete;
    DontCopyMe& operator= (const DontCopyMe& a) = delete;
}
```

## 条款7：为多态基类声明virtual析构函数

1. 给base classes一个virtual析构函数只适用于polymorphic base classes，这种类设计目的便是通过基类接口处理derived classes对象。
2. 并非所有base classes设计目的是为了多态用途，这些classes不需要virtual析构函数。
3. polymorphic base classes应该声明一个析构函数，如果class带有任何virtual函数，它就应该拥有一个virtual析构函数。
4. classes的设计目的如果不是作为base class使用，或者不是为了具备polymorphically，就不该声明virtual析构函数。

## 条款8：别让异常逃离析构函数

1. 析构函数绝对不要吐出异常，如果一个被析构函数调用的函数可能抛出异常，析构函数应该捕捉任何异常然后吞下他们 or 结束程序。
2. 如果客户需要对某个操作函数运行期间抛出的异常作出反应，那么class需要提供一个普通函数供用户使用处理异常。

## 条款9：绝不在构造和析构过程中调用virtual函数

1. 在derived class对象的base class构造期间，对象的类型是base class而不是derived class。于是构造过程中调用的是基类的virtual函数。
2. 无法使用virtual函数从base classes向下调用，构造期间可以令derived classes将必要的构造信息向上传递给base class的构造函数。

```c++
class Transaction {
public:
    explicit Transaction(const std::string& logInfo);
    void LogTransaction(const std::string& logInfo) const;
    ...
};

Transaction::Transaction(const std::string& logInfo) {
    LogTransaction(logInfo);                           // 更改为了非虚函数调用
}

class BuyTransaction : public Transaction {
public:
    BuyTransaction(...)
        : Transaction(CreateLogString(...)) { ... }    // 将信息传递给基类构造函数
    ...

private:
    static std::string CreateLogString(...);
}
```

## 条款10：令operator= 返回一个reference to *this

令赋值操作符返回一个`reference to *this`，虽然并不强制执行此条款，但为了实现连锁赋值，大部分时候应该这样做。

## 条款11：在 operator= 中处理“自我赋值”

自我赋值是合法的操作，但在一些情况下可能会导致意外的错误，例如在复制堆上的资源时：

```c++
Widget& operator+=(const Widget& rhs) {
    delete pRes;                          // 删除当前持有的资源
    pRes = new Resource(*rhs.pRes);       // 复制传入的资源
    return *this;
}

// 导致访问到已删除的数据
```

最简单的解决方法是在执行后续语句前先进行**证同测试（Identity test）**：`if (this == &rhs) return *this;`

还有一种取巧的做法是使用 copy and swap 技术，这种技术聪明地利用了栈空间会自动释放的特性，这样就可以通过析构函数来实现资源的释放：

```c++
Widget& operator=(const Widget& rhs) {
    Widget temp(rhs);
    std::swap(*this, temp);
    return *this;
}
// 或者是按值传参
Widget& operator=(Widget rhs) {
    std::swap(*this, rhs);
    return *this;
}
```

## 条款 12：复制对象时勿忘其每一个成分

决定手动实现拷贝构造函数或拷贝赋值运算符时，忘记复制任何一个成员都可能会导致意外的错误。

当使用继承时，**继承自基类的成员往往容易忘记在派生类中完成复制**，如果你的基类拥有拷贝构造函数和拷贝赋值运算符，应该记得调用它们：

```c++
class PriorityCustomer : public Customer {
public:
    PriorityCustomer(const PriorityCustomer& rhs);
    PriorityCustomer& operator=(const PriorityCustomer& rhs);
    ...

private:
    int priority;
}

PriorityCustomer::PriorityCustomer(const PriorityCustomer& rhs)
    : Customer(rhs),                // 调用基类的拷贝构造函数
      priority(rhs.priority) {
    ...
}

PriorityCustomer::PriorityCustomer& operator=(const PriorityCustomer& rhs) {
    Customer::operator=(rhs);       // 调用基类的拷贝赋值运算符
    priority = rhs.priority;
    return *this;
}
```

注意，**不要尝试在拷贝构造函数中调用拷贝赋值运算符**，或在拷贝赋值运算符的实现中调用拷贝构造函数，一个在初始化时，一个在初始化后，它们的功用是不同的。

# 资源管理

## 条款13：以对象管理资源

对于传统的堆资源管理，我们需要使用成对的`new`和`delete`，这样若忘记`delete`就会造成内存泄露。因此，我们应尽可能以对象管理资源，并采用**RAII（Resource Acquisition Is Initialize，资源取得时机便是初始化时机）**，让析构函数负责资源的释放。

注：RAII——对资源申请、释放这种成对的操作的封装，通过这种方式实现在局部作用域内申请资源然后销毁资源。

可以使用`std::unique_tpr`等智能指针。

## 条款14：在资源管理类中小心拷贝行为

当RAII对象被复制，会发生什么事情？

1. 禁止复制：见条款6。

2. 对底层资源祭出“引用计数法”：正如`std::shared_ptr`所做的那样，每一次复制对象就使引用计数+1，每一个对象离开定义域就调用析构函数使引用计数-1，直到引用计数为0就彻底销毁资源。

   ```c++
   class Lock {
   public:
       explicit Lock(Mutex* pm) : mutexPtr(pm, unlock) {
           lock(mutexPtr.get());
       }
   private:
       std::shared_ptr<Mutex> mutexPtr;
   };
   ```

3. 复制底层资源：在复制对象的同时复制底层资源的行为又被称作**深拷贝（Deep copying）**，例如在一个对象中有一个指针，那么在复制这个对象时就不能只复制指针，也要复制指针所指向的数据。

4. 转移底层资源的所有权：和`std::unique_ptr`的行为类似，永远保持只有一个对象拥有对资源的管理权，当需要复制对象时转移资源的管理权。

## 条款15：在资源管理类中提供对原始资源的访问

```c++
std::unique_ptr<Investment> pInv(createInvestment());

int daysHeld(const Investment* pi);

daysHeld(pInv);	//COMPILE ERROR
daysHeld(pInv.get()); 	//OK
```

当我们在设计自己的资源管理类时，也要考虑在提供对原始资源的访问时，是使用显式访问还是隐式访问的方法，还是两者皆可。

```c++
class Font {
public:
    FontHandle get() const { return handle; }       // 显式转换函数
    operator FontHandle() const { return handle; }  // 隐式转换函数

private:
    FontHandle handle;
};
int daysHeld(const FontHandle fh);
int main() {
    Font font;
    font.get();	// 显式get
    daysHeld(font);	// 隐式get
    return 0;
}
```

## 条款16：成对使用 new 和 delete 时要采用相同形式

使用`new`来分配单一对象，使用`new[]`来分配对象数组，必须明确它们的行为并不一致，分配对象数组时会额外在内存中记录“数组大小”，而使用`delete[]`会根据记录的数组大小多次调用析构函数，使用`delete`则仅仅只会调用一次析构函数。对于单一对象使用`delete[]`其结果也是未定义的，程序可能会读取若干内存并将其错误地解释为数组大小。

## 条款17：以独立语句将 newed 对象置入智能指针

现在使用`std::make_unique<Class>()`。

```c++
auto pUniqueInv = std::make_unique<Investment>();    // since C++ 14
auto pSharedInv = std::make_shared<Investment>();    // since C++ 11
```

# 设计与声明

## 条款18：让接口容易被正确使用，不易被误用

1. 好的接口很容易被正确使用，不易被误用。你应在在你的所有接口中努力达成这些性质。

   ```c++
   // 三个参数类型相同的函数容易造成误用
   Data::Data(int month, int day, int year) { ... }
   
   // 通过适当定义新的类型加以限制，降低误用的可能性
   Data::Data(const Month& m, const Day& d, const Year& y) { ... }
   ```

2. “促进正确使用”的办法包括接口的一致性，以及与**内置类型的行为兼容**。

3. “阻止误用”的办法包括建立新类型、限制类型上的操作，束缚对象值，以及消除客户的资源管理责任(智能指针)。

4. 尽量使用智能指针，避免跨DLL的 new 和 delete，使用智能指针自定义删除器来解除互斥锁（mutexes）。

   ```c++
   // 实现createInvestment并使它返回一个tr1::shared_ptr，并带有删除器
   std::tr1::shared_ptr<Investment> createInvestment() {
       std::tr1::shared_ptr<Investment> retVal(static_cast<Investment*>(0), getRidOfInvestment);
       retVal = ...;	// 指向正确对象
       return retVal;
   }
   
   // 删除器是一个单参数，接受类型指针，无返回值
   // 函数式删除器
   void Deleter(Connection *connection){
       close(connection);
       delete connection;
   }
   
   int main(){
       // 新建管理连接Connection的智能指针
       shared_ptr<Connection> sp(new Connection("shared_ptr"), Deleter);
       unique_ptr<Connection, decltype(Deleter)*> up(new Connection("unique_ptr"), Deleter);
   }
   // shared_ptr在使用的时候，只需要把函数式删除器的指针传给构造函数就行；
   // 而unique_ptr还用增加一个模板参数decltype(Deleter)*，这是shared_ptr和shared_ptr的不同点之一
   // （注意：unique_ptr的第二个模板参数是指针）。
   
   // 或是lambda表达式
   int main(){
       auto DeleterLambda=[](Connection *connection){
           close(connection);
           delete connection;
       };
       // 新建管理连接Connection的智能指针
       shared_ptr<Connection> sp(new Connection("shared_ptr"), DeleterLambda);
       unique_ptr<Connection, decltype(DeleterLambda)> up(new Connection("unique_ptr"), DeleterLambda);
   }
   ```

## 条款19：设计 class 犹如设计 type

几乎在设计每一个 class 时，都要面对如下问题：

1. **新 type 对象应该如何被创建和销毁？** 这会影响到类中构造函数、析构函数、内存分配和释放函数（`operator new`，`operator new[]`，`operator delete`，`operator delete[]`）的设计。

2. **对象的初始化和赋值该有什么样的差别？** 这会影响到构造函数和拷贝赋值运算之间行为的差异。

3. **新 type 的对象如果被按值传递，意味着什么？** 这会影响到拷贝构造函数的实现。

4. **什么是新 type 的合法值？** 你的类中的成员函数必须对类中成员变量的值进行检查，如果不合法就要尽快解决或明确地抛出异常。

5. **你的新 type 需要配合某个继承图系吗？** 你的类是否受到基类设计地束缚，是否拥有该覆写地虚函数，是否允许被继承（若不想要被继承，应该声明为`final`）。

6. **什么样的运算符和函数对此新 type 而言是合理的？** 这会影响到你将为你的类声明哪些函数和重载哪些运算符。

7. **什么样的标准函数应该被驳回？** 这会影响到你将哪些标准函数声明为`= delete`。

8. **谁该取用新 type 的成员？** 这会影响到你将类中哪些成员设为 public，private 或 protected，也将影响到友元类和友元函数的设置。

9. **什么是新 type 的“未声明接口”？** 为未声明接口提供效率、异常安全性以及资源运用上的保证，并在实现代码中加上相应的约束条件。

10. **你的新 type 有多么一般化？** 如果你想要一系列新 type 家族，应该优先考虑模板类。

## 条款20：宁以按常引用传参替换按值传参

当使用按值传参时，程序会调用对象的拷贝构造函数构建一个在函数内作用的局部对象，这个过程的开销可能会较为昂贵。对于任何用户自定义类型，使用按常引用传参是较为推荐的：

`bool ValidateStudent(const Student& s);`

因为没有任何新对象被创建，这种传参方式不会调用任何构造函数或析构函数，所以效率比按值传参高得多。

使用按引用传参也可以避免**对象切片（Object slicing）** 的问题，参考以下例子：

```c++
class Window {
public:
    ...
    std::string GetName() const;
    virtual void Display() const;
};

class WindowWithScrollBars : public Window {
public:
    virtual void Display() const override;
};
```

此处一个`WindowWithScrollBars`类继承自`Window`基类。

```c++
void PrintNameAndDisplay(Window w) {    // 按值传参，会发生对象切片
    std::cout << w.GetName();
    w.Display();
}
```

此处在传参时，调用了基类`Window`的拷贝构造函数而非派生类的拷贝构造函数，因此在函数种使用的是一个`Window`对象，调用虚函数时也只能调用到基类的虚函数`Window::Display`。

由于按引用传递不会创建新对象，这个问题就能得到避免：

```c++
void PrintNameAndDisplay(const Window& w) { // 参数不会被切片
    std::cout << w.GetName();
    w.Display();
}
```

也并非永远都使用按引用传参，对于内置类型、STL的迭代器和函数对象，我们认为使用按值传参是比较合适的。

## 条款 21：必须返回对象时，别妄想返回其引用

1. 绝不要返回pointer或reference指向一个local stack对象(会被释放导致指针悬空)，或返回reference指向一个heap-allocated对象(函数体内new，返回reference，无法delete！)，或返回pointer或reference指向一个local static对象而有可能同时需要多个这样的对象(会变成一样的！)。

2. 尽管返回对象会调用拷贝构造函数产生开销，但这开销比起出错而言微不足道。

## 条款 22：将成员变量声明为 private

出于对封装性的考虑，应该尽可能地隐藏类中的成员变量，并通过对外暴露函数接口来实现对成员变量的访问：

```c++
class AcessLevels {
public:
    int GetReadOnly() const { return readOnly; }
    void SetReadWrite(int value) { readWrite = value; }
    int GetReadWrite() const { return readWrite; }
    void SetWriteOnly(int value) { writeOnly = value; }

private:
    int noAccess;
    int readOnly;
    int readWrite;
    int writeOnly;
};
```

通过为成员变量提供 getter 和 setter 函数，我们就能避免客户做出写入只读变量或读取只写变量这样不被允许的操作。

将成员变量隐藏在函数接口的背后，可以为“所有可能的实现”提供弹性。例如这可使得在成员变量被读或写时轻松通知其它对象，可以验证类的约束条件以及函数的提前和事后状态，可以在多线程环境中执行同步控制……

`protected`和`public`一样，都不该被优先考虑。**假设我们有一个public成员变量，最终取消了它，那么所有使用它的客户代码都将被破坏**；假设我们有一个protected成员变量，最终取消了它，那么所有使用它的**派生类都将被破坏**。

综合以上讨论，在类中应当将成员变量优先声明为 private。

## 条款23：宁以非成员、非友元函数替换成员函数

```c++
class WebBrowser {
public:
    ...
    void ClearCache();
    void ClearHistory();
    void RemoveCookies();
    ...
};
```

如果想要一次性调用这三个函数，那么需要额外提供一个新的函数：

```c++
void ClearEverything(WebBrowser& wb) {
    wb.ClearCache();
    wb.ClearHistory();
    wb.RemoveCookies();
}
```

注意，虽然成员函数和非成员函数都可以完成我们的目标，但此处更建议使用非成员函数，这是为了遵守一个原则：**越少的代码可以访问数据，数据的封装性就越强**。此处的`ClearEverything`函数仅仅是调用了`WebBrowser`的三个public成员函数，而并没有使用到`WebBrowser`内部的private成员，因此没有必要让其也拥有访问类中private成员的能力。

这个原则对于友元函数也是相同的，因为友元函数和成员函数拥有相同的权力，所以在能使用非成员函数完成任务的情况下，就不要使用友元函数和成员函数。

如果你觉得一个全局函数并不自然，也可以考虑将`ClearEverything`函数放在工具类中充当静态成员函数，或与`WebBrowser`放在同一个命名空间中：

```c++
namespace WebBrowserStuff {
    class WebBrowser { ... };
    void ClearEverything(WebBrowser& wb) { ... }
}
```

## 条款 24：若所有参数皆需类型转换，请为此采用非成员函数

现在我们手头上拥有一个`Rational`类，并且它可以和`int`隐式转换：

```cpp
class Rational {
public:
    Rational(int numerator = 0, int denominator = 1);
    ...
};
```

当然，我们需要重载乘法运算符来实现`Rational`对象之间的乘法：

```cpp
class Rational {
public:
    ...
    const Rational operator*(const Rational& rhs) const;
};
```

将运算符重载放在类中是行得通的，至少对于`Rational`对象来说是如此。但当我们考虑混合运算时，就会出现一个问题：

```c++
Rational oneEight(1, 8);
Rational oneHalf(1, 2);
Rational result = oneHalf / oneEight;

result = oneHalf * 2;    // 正确
result = 2 * oneHalf;    // 报错
```

假如将乘法运算符写成函数形式，错误的原因就一目了然了：

```cpp
result = oneHalf.operator*(2);    // 正确
result = 2.operator*(oneHalf);    // 报错
```

在调用`operator*`时，`int`类型的变量会隐式转换为`Rational`对象，因此用`Rational`对象乘以`int`对象是合法的，但反过来则不是如此。

所以，为了避免这个错误，我们应当将运算符重载放在类外，作为非成员函数：

```cpp
const Rational operator*(const Rational& lhs, const Rational& rhs);
```

## 条款 25：考虑写出一个不抛异常的swap函数

两个类：

```c++
class WidgetImpl {
public:
    ...;
private:
    int a, b, c;
    std::vector<int> v;
    ...;
};

class Widget {
public:
        Widget(const Widget& rhs);
    Widget& operator=(const Widget& rhs) {
        ...;
        *pImpl = *(rhs.pImpl);	// 调用拷贝构造函数
        ...;
    }
    ...;
private:
    WidgetImpl* pImpl;	// 指向对象内含的Widget数据
};
```

要置换两个Widget的对象值，我们唯一需要的就是对换pImpl指针，但swap算法并不知道。

```c++
// 特化版本1：(无法编译)
namespace std {
    template<>
    void swap<Widget> (Widget& a, Widget& b) {
        swap(a.pImpl, b.pImpl);	// 私有变量无法访问
    }
}
// 特化版本2：加入成员函数
class Widget {
public:
    void swap(Wight& other) {
        using std::swap;
        swap(pImpl, other.Impl);
    }
    ...
}
namespace std {
    template<>
    void swap<Widget> (Widget& a, Widget& b) {
        a.swap(b);
    }
}
```

然而若`Widget`和`WidgetImpl`是类模板，情况就没有这么简单了，因为 C++ 不支持函数模板偏特化，所以只能使用重载的方式：

```c++
namespace std {
    template<typename T>
    void swap(Widget<T>& a, Widget<T>& b) {
        a.swap(b);
    }
}
```

但很抱歉，这种做法是被 STL 禁止的，因为这是在试图向 STL 中添加新的内容，所以我们只能退而求其次，在其它命名空间中定义新的swap函数：

```cpp
namespace WidgetStuff {
    ...
    template<typename T>
    class Widget { ... };
    ...
    template<typename T>
    void swap(Widget<T>& a, Widget<T>& b) {
        a.swap(b);
    }
}
```

我们希望在对自定义对象进行操作时找到正确的swap函数重载版本，这时候如果再写成`std::swap`，就会强制使用 STL 中的swap函数，无法满足我们的需求，因此需要改写成：

```cpp
using std::swap;
swap(obj1, obj2);
```

这样，C++ 名称查找法则能保证我们优先使用的是自定义的swap函数而非 STL 中的swap函数。

**C++名称查找法则**：编译器会从使用名字的地方开始向上查找，由内向外查找各级作用域（命名空间）直到全局作用域（命名空间），找到同名的声明即停止，若最终没找到则报错。 函数匹配优先级：普通函数 > 特化函数 > 模板函数
