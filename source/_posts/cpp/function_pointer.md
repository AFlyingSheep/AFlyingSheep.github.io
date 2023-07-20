---
title: C++函数指针
date: 2023-2-27 13:46:08
tags:
- [CS]
- [C/C++]
categories: 
- [CS]
mathjax: true
description: 函数指针是指指向函数而非指向对象的指针。像其他指针一样，函数指针也指向某个特定的类型。函数类型由其返回类型以及形参表确定,而与函数名无关.
top_img: 
cover: /image/covers/funcp.png
---
函数指针是指指向函数而非指向对象的指针。像其他指针一样，函数指针也指向某个特定的类型。函数类型由其返回类型以及形参表确定， 而与函数名无关：

```c++
// pf points to function returning bool that takes two const string references
bool (*pf)(const string &, const string &);
```

这个语句将 pf 声明为指向函数的指针，它所指向的函数带有两个 const string& 类型的形参和 bool 类型的返回值。

*pf 两侧的圆括号是必需的：

```c++
// declares a function named pf that returns a bool*
bool *pf(const string &, const string &);
```

## 用 typedef 简化函数指针的定义

函数指针类型相当地冗长。使用 typedef 为指针类型定义同义词，可将函数指针的使用大大简化：

```c++
typedef bool (*cmpFcn)(const string &, const string &);
```

该定义表示 cmpFcn 是一种指向函数的指针类型的名字。该指针类型为“指向返回 bool 类型并带有两个 const string 引用形参的函数的指针”。在要使用这种函数指针类型时，只需直接使用 cmpFcn 即可，不必每次都把整个类型声明全部写出来。

## 指向函数的指针的初始化和赋值

在引用函数名但又没有调用该函数时，函数名将被自动解释为指向函数的指针。假设有函数：

```c++
// compares lengths of two strings
bool lengthCompare(const string &, const string &);
```

除了用作函数调用的左操作数以外，对 lengthCompare 的任何使用都被解释为如下类型的指针：

```c++
bool (*)(const string &, const string &);
```

此时，直接引用函数名等效于在函数名上应用取地址操作符：

```c++
cmpFcn pf1 = lengthCompare;
cmpFcn pf2 = &lengthCompare;
```

函数指针只能通过同类型的函数或函数指针或 0 值常量表达式进行初始化或赋值。

将函数指针初始化为 0，表示该指针不指向任何函数。

指向不同函数类型的指针之间不存在转换。

## 通过指针调用函数

指向函数的指针可用于调用它所指向的函数。可以不需要使用解引用操作符，直接通过指针调用函数：

```c++
cmpFcn pf = lengthCompare;
lengthCompare("hi", "bye"); // direct call
pf("hi", "bye"); // equivalent call: pf1 implicitly dereferenced
(*pf)("hi", "bye"); // equivalent call: pf1 explicitly dereferenced
```

如果指向函数的指针没有初始化，或者具有 0 值，则该指针不能在函数调用中使用。只有当指针已经初始化，或被赋值为指向某个函数，方能安全地用来调用函数。

## 函数指针的形参

函数的形参可以是指向函数的指针。这种形参可以用以下两种形式编写：

```c++
void useBigger(const string &, const string &,bool(const string &, const string &));
void useBigger(const string &, const string &,bool (*)(const string &, const string &));
```

## 返回指向函数的指针

函数可以返回指向函数的指针，但是，正确写出这种返回类型相当不容易：

```c++
int (*ff(int))(int*, int);
```

它是一个指向函数的指针，所指向的函数返回 int 型并带有两个分别是int* 型和 int 型的形参。

使用 typedef 可使该定义更简明易懂：

```c++
typedef int (*PF)(int*, int);
PF ff(int); // ff returns a pointer to function
```

允许将形参定义为函数类型，但函数的返回类型则必须是指向函数的指针，而不能是函数。

具有函数类型的形参所对应的实参将被自动转换为指向相应函数类型的指针。但是，当返回的是函数时，同样的转换操作则无法实现：

```c++
// func is a function type, not a pointer to function!
typedef int func(int*, int);
void f1(func); // ok: f1 has a parameter of function type
func f2(int); // error: f2 has a return type of function type
func *f3(int); // ok: f3 returns a pointer to function type
```

## 指向重载函数的指针

C++ 语言允许使用函数指针指向重载的函数：

```c++
extern void ff(vector<double>);
extern void ff(unsigned int);
```

指针的类型必须与重载函数的一个版本精确匹配。如果没有精确匹配的函数，则对该指针的初始化或赋值都将导致编译错误：

```c++
// error: no match: invalid parameter list
void (*pf2)(int) = &ff;
// error: no match: invalid return type
double (*pf3)(vector<double>);
pf3 = &ff;
```

## decltype用于函数指针类型

```c++
string::size_type sumLength(const string&,const string&);
decltype(sumLength) *getFcn(const string&)
```

使用decltype得到的是某个函数类型而非指针类型，因此需要显示的加上指针。 
