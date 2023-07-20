---
title: C++虚函数表的位置--从内存的角度
date: 2023-4-5 11:37:06
tags:
- [C++]
categories: 
- [C++]
mathjax: true
description: 从内存角度分析虚函数表。
top_img: 
cover: /image/covers/virtual.png
---

# 零成本抽象

对于如下类，大小为4字节，也就是一个int的大小，跑这个类如同跑一个单独的int

```c++
class A {
public:
    int x;
};
```

类这个概念，只存在于编译时期。

也就是，我们可以写出修改类中的私有变量的代码（因为，私有这个东西，只在编译时期中存在）：

```c++
class A {
private:
    int x;
public:
    int getx() { return x; }
};
int main()
{
    cout << sizeof(A) << endl;
    A a;
    int* p = (int*)&a;
    *p = 114514;
    cout << a.getx() << endl;
    return 0;
}

/*
	输出：
	4
	114514
*/
```

这个时候我们发现，**函数是不占空间的**。

我们写出一个继承：

```c++
class A {
public:
    int x, y;
    void show() { cout << "show" << endl; }
};
class B :public A {
public:
    int z;
};
int main(){
    cout << sizeof(A) << endl;
    cout << sizeof(B) << endl;
    return 0;
    
    printf("%p\n", &A::show);
	printf("%p\n", &B::show);
}
/*
	输出：
	8
	12
	00007FF75D8A152D
	00007FF75D8A152D
*/
```

![1](/image/vt/1.png)

两个类共享一个show，这个show不会占用类的空间(全局数据区存放全局变量，静态数据和常量；所有**类成员函数和非成员函数代码存放在代码区**；为运行函数而分配的局部变量、函数参数、返回数据、返回地址等存放在栈区；余下的空间都被称为堆区)

# 带有虚函数的类

```c++
class A1 {
public:
    virtual void a() { cout << "A1 a()" << endl; }
    virtual void b() { cout << "A1 b()" << endl; }
    virtual void c() { cout << "A1 c()" << endl; }
};

class A2 {
public:
    virtual void a() { cout << "A2 a()" << endl; }
    virtual void b() { cout << "A2 b()" << endl; }
    virtual void c() { cout << "A2 c()" << endl; }
    int x, y;
};
```

输出发现A1大小为8字节，A2大小为16字节。也就是，只要有虚函数，无论多少个，都会增加8的大小(64位系统)，说明增加了指针。

此时内存模型：

![1](/image/vt/2.png)

探索一下虚函数表：

![1](/image/vt/3.png)

```c++
class A {
public:
    virtual void a() { cout << "A a()" << endl; }
    virtual void b() { cout << "A b()" << endl; }
    virtual void c() { cout << "A c()" << endl; }
    int x, y;
};

int main(){
    typedef long long u64;
    typedef void(*func)();
    
    A a;
    // 指向虚函数指针
    u64* p = (u64*)&a;
    // 指向虚函数表
    u64* arr = (u64*)*p;
   
    // 调用虚函数
    func fa = (func)arr[0];
    func fb = (func)arr[1];
    func fc = (func)arr[2];
    fa(); fb(); fc();
    return 0;
}

/*
	输出：
	A a()
	A b()
	A c()
*/
```

对于A的实例化，虚函数指针都是指向同一块的(指向虚函数表)。

派生一个B：

```c++
class B :public A {
public:
    int z;
    virtual void b() { cout << "B b()" << endl; }
};
```

![1](/image/vt/4.png)

# 待补充

多继承情况下对象和虚表的布局、thunk这种编译器魔法

# 总结：

1. 每个类,只要含有虚函数,new出来的对象就包含一个虚函数指针,指向这个类的虚函数表(这个虚函数表一个类用一张)

2. 子类继承父类,会形成一个新的虚函数表,但是虚函数的实际地址还是用的父类的,如果子类重写了某个虚函数,那么子类的虚函数表中存放的就是重写的虚函数的地址

3. 不同类之间可以通过强制转型调用其他类的虚函数

注：转载自[C++虚函数表的位置——从内存的角度 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/563418849)

