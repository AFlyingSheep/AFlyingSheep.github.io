---
title: C++：CRTP与侵入式链表的一种实现
date: 2023-5-13 12:06:06
tags:
- [CS]
- [C/C++]
categories: 
- [CS]
- [C/C++]
mathjax: true
description: CRTP是Curiously Recurring Template Pattern的缩写，中文可以翻成奇异递归模板，它是通过将子类类型作为模板参数传给基类的一种模板的使用技巧.
top_img: 
cover: /image/covers/CRTP.jpg
---

# 介绍

CRTP是Curiously Recurring Template Pattern的缩写，中文可以翻成奇异递归模板，它是通过将子类类型作为模板参数传给基类的一种模板的使用技巧。

# 静态多态

最近在做关于神威上基于Codelet模型实现的数据流编程相关项目，以里面的创建`Task`举例说明：

```c++
#include <cstdio>

template <typename T>
class Task {
public:
    void execute() {
        static_cast<T*>(this)->execute();
    }
};

class Mytask : public Task<Mytask> {
public:
    void execute() {
        printf("This is my task.\n");
    }
};

// 类似于运行时系统执行
template <typename T>
void execute_now(Task<T> &task) {
    task.execute();
}

int main(int argc, char** argv) {
    Mytask mt;
    // 运行时系统运行提交的任务
    execute_now(mt);
}
```

优点：CRTP使得类具有虚函数的效果，同时也没有虚函数的调用开销(查虚函数表)。

缺点：无法进行动态绑定、代码可读性降低、实例化后的code size可能变大。

# 代码复用

以`enable_shared_from_this`举例。如果使用智能指针管理类，在类的成员函数中需要返回对外部的`this`指针，如果直接返回，保存到外部的某个变量中，变量便并不知道什么时候可以使用(可能已经被析构)。故需要返回智能指针，让类继承自`enable_shared_from_this`并调用父类的`shared_from_this()`方法。

```c++
// source code - enable shared from this 
// select key part
template<class _Ty>
	class enable_shared_from_this
	{	// provide member functions that create shared_ptr to this
public:
	shared_ptr<_Ty> shared_from_this()
		{	// return shared_ptr
		return (shared_ptr<_Ty>(_Wptr));
		}

	shared_ptr<const _Ty> shared_from_this() const
		{	// return shared_ptr
		return (shared_ptr<const _Ty>(_Wptr));
		}
private:
	template<class _Ty1,
		class _Ty2>
		friend void _Do_enable(
			_Ty1 *,
			enable_shared_from_this<_Ty2>*,
			_Ref_count_base *);
	weak_ptr<_Ty> _Wptr;
	};
```

使用CRTP，使得每个继承它的子类都可以调用`shared_from_this`方法，并根据传入模板类型返回不同的智能指针。

# 实例化多套基类静态变量和方法

对于多个类含有相同的静态变量或方法，可以直接在基类进行，而无需在每个类中均定义一次：

```c++
#include <cstdio>
#include <memory>

template <typename T>
class CountClass {
public:
    static void getCnt() {
        printf("Cnt count: %d.\n", cnt);
    }
protected:
    static int cnt;
};

template <typename T> int CountClass<T>::cnt = 0;

class C1 : public CountClass<C1> {
public:
    C1() {cnt++;}
};

class C2 : public CountClass<C2> {
public:
    C2() {cnt++;}
};


int main(int argc, char** argv) {
    C1 c11, c12;
    C2 c21;
    CountClass<C1>::getCnt();
    CountClass<C2>::getCnt();
}

// result:
// Cnt count: 2.
// Cnt count: 1.
```

# 基于CRTP实现的侵入式链表

没怎么看明白侵入式链表和非侵入式的区别，需要以后细细琢磨，先把CRTP弄明白了。

第一版：`ListNode`作为链表基类，使用CRTP `MyList`实现链表数据部分，`ListPtr`作为基类的内部类用于操作指向基类的指针。

缺点：没有资源释放！

```c++
#include <cstdio>
#include <memory>
#include <iostream>

// CRTP, T -> Node with data
template <typename T>
class ListNode {
public:
    T* _next;
    ListNode<T>(T* n = nullptr) : _next(n) {}
    class ListPtr {
        T* p;
    public:
        ListPtr(T* t = nullptr) : p(t) {};
        T* operator->() {return p;}
        ListPtr &operator=(ListPtr &rhs) {p = rhs.p;}
        template <typename ... Args>
        void push_back(Args&& ... args) {
            auto *ptr = new T(std::forward<Args>(args)...);
            ptr->_next = p->_next;
            p->_next = ptr;
        }
        void next() {p = p->_next;}
        bool has_next() {return p->_next != nullptr;}
        bool is_end() {return p == nullptr;}
        void operator++(int) {
            next();
        }
    };
};

class PairList : public ListNode<PairList> {
    int val1, val2;
public:
    using ptr = ListNode<PairList>::ListPtr;
    PairList(int ival1, int ival2) : val1(ival1), val2(ival2) {}
    void getValue() {
        std::cout << val1 << ":" << val2 << std::endl;
    }
};

template <typename T, typename ... Args>
auto make_list(Args&& ... args) {
    auto ptr = typename ListNode<T>::ListPtr(new T(std::forward<Args>(args)...)); 
    return ptr;
}

int main() {
    PairList::ptr p = make_list<PairList>(1, 2);
    for (int i = 0; i < 10; i++) {
        p.push_back(i, i + 1);
    }
    for (auto it = p; it.has_next(); it++) {
        it->getValue();
    }
    return 0;
}
```

第二版：加入Unique_ptr管理内存释放。

```c++
#include <cstdio>
#include <memory>
#include <iostream>

// CRTP, T -> Node with data
template <typename T>
class ListNode {
public:
    T* _next;
    ListNode<T>(T* n = nullptr) : _next(n) {}

    class Unique_ListPtr {
        T* data = nullptr;
        void clean(T* rhs) {
            if (rhs->_next != nullptr) {
                clean(rhs->_next);
            }
            delete rhs;
        }
    public:
        Unique_ListPtr(): data(nullptr) {}
        explicit Unique_ListPtr(T* rhs) : data(rhs) {}
        Unique_ListPtr(const Unique_ListPtr &) = delete;
        Unique_ListPtr(Unique_ListPtr && rhs) {
            data = rhs.data;
            rhs.data = nullptr;
        }
        ~Unique_ListPtr() {
            if (data) clean(data);
            std::cout << "Unique ptr clean." << std::endl;
        }
        Unique_ListPtr& operator=(const Unique_ListPtr &) = delete;
        void operator=(Unique_ListPtr &rhs) {
            std::swap(rhs, data);
        }
        T* get() const {return data;}
    };

    class ListPtr {
        T* p;
    public:
        ListPtr(const Unique_ListPtr &ul) : p(ul.get()) {};
        T* operator->() {return p;}
        ListPtr &operator=(ListPtr &rhs) {p = rhs.p;}
        ListPtr &operator=(const Unique_ListPtr &rhs) {p = rhs.get();}
        template <typename ... Args>
        void push_back(Args&& ... args) {
            auto *ptr = new T(std::forward<Args>(args)...);
            ptr->_next = p->_next;
            p->_next = ptr;
        }
        void next() {p = p->_next;}
        bool has_next() {return p->_next != nullptr;}
        bool is_end() {return p == nullptr;}
        void operator++(int) {
            next();
        }
    };
};

class PairList : public ListNode<PairList> {
    int val1, val2;
public:
    using ptr = ListNode<PairList>::ListPtr;
    PairList(int ival1, int ival2) : val1(ival1), val2(ival2) {}
    void getValue() {
        std::cout << val1 << ":" << val2 << std::endl;
    }
};

template <typename T, typename ... Args>
auto make_list(Args&& ... args) {
    auto ptr = typename ListNode<T>::Unique_ListPtr(new T(std::forward<Args>(args)...)); 
    return ptr;
}

int main() {
    // error: Unique ptr has been cleaned after make_list called.
    // PairList::ptr p = make_list<PairList>(1, 2);
    auto uni_begin = make_list<PairList>(0, 1);
    PairList::ptr p = uni_begin;
    for (int i = 0; i < 3; i++) {
        p.push_back(i, i + 1);
    }
    for (auto it = p; it.has_next(); it++) {
        it->getValue();
    }
    return 0;
}
```

