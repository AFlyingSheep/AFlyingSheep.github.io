---
title: C++：模板类和静态成员变量
date: 2023-1-8 16:57:06
tags:
- [C++]
categories: 
- [C++]
mathjax: true
description: 当一个模板类产生不同的类时，每个类产生的对象共享static变量，静态成员变量作用于类层面
top_img: 
cover: /image/cpp/cover_2.png
---

# C++ 模板类和静态成员变量

当一个模板类产生不同的类时，**每个类产生的对象共享static变量**，静态成员变量作用于类层面。即类间不共享，类内共享。

```c++
#include <bits/stdc++.h>

template <typename size_type>
class hello {
public:
	static int s_a;
	void add() {
		s_a++;
	}
}; 

// 模板类中静态变量的定义
template <class size_type> int hello<size_type>::s_a = 0;

int main () {
	hello<int> h3;
	hello<char> h4; 

	h3.add();
	h4.add();
	h4.add();
	
	std::cout << hello<int>::s_a << std::endl;
	std::cout << hello<char>::s_a << std::endl;
	
    /*
    out:
    1
    2
    */
}
```

