---
title: C++11：tuple
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
cover: /image/covers/tuple.png
---

# tuple

 std::tuple是C++11新标准引入的一个类模板，又称为元组，是一个固定大小的异构值集合，由std::pair泛化而来。pair可以看作是tuple的一种特殊情况，成员数t目限定为两个。tuple可以有任意个成员数量，但是每个确定的tuple类型的成员数目是固定的。

 从概念上讲，它们类似于C的结构体，但是不具有命名的数据成员，我们也可以把他当做一个通用的结构体来用，不需要创建结构体又获取结构体的特征，在某些情况下可以取代结构体使程序更简洁，直观。

 tuple 的应用场景很广泛，例如当需要存储多个不同类型的元素时，可以使用 tuple；当函数需要返回多个数据时，可以将这些数据存储在 tuple 中，函数只需返回一个 tuple 对象即可。

# tuple基本用法

## tuple对象的创建和初始化

1. 类的构造函数

   ```c++
   #include <iostream>
   #include <tuple>
   
   int main() {
       std::tuple<std::string, int> t1;    //无参构造
       std::tuple<std::string, int> t2(t1);//拷贝构造
       std::tuple<std::string, int> t3(std::make_tuple("Lily", 1));
       std::tuple<std::string, long> t4(t3);   //隐式类型转换构造的左值方式
       std::tuple<std::string, int> t5("Mike", 2); //初始化列表构造的右值方式
       std::tuple<std::string, int> t6(std::make_pair("Jack",3));  //将pair对象转换为tuple对象
       return 0;
   }
   ```

2. make_tuple()函数：make_tuple() 函数以模板的形式定义在头文件中，功能是创建一个 tuple 右值对象（或者临时对象）。

   ```c++
   std::tuple<std::string, double ,int> t1 = std::make_tuple("Jack", 66.6, 88);
   auto t2 = std::make_tuple(1, "Lily", 'c');
   
   //t2的类型实际是std::tuple<int, const char *, char>
   ```

## 获取tuple的值

1. std::get(std::tuple)

   ```c++
   #include <iostream>
   #include <tuple>
   
   int main() {
       auto t1 = std::make_tuple(1, "PI", 3.14);
       std::cout << '(' << std::get<0>(t1) << ',' << std::get<1>(t1) <<
               ',' << std::get<2>(t1) <<  ')' << '\n';
       return 0;
   }
   /*
   output: (1,PI,3.14)
   */
   
   int main() {
   	std::tuple<std::tuple<int, int>, int, int> t;
   	t = std::make_tuple(std::tuple<int, int>(2, 1), 1, 1); 
   	
   	std::cout << std::get<0>(std::get<0>(t));
   }
   
   /*
   	output: 2
   */
   ```

由于get的特性，**tuple不支持迭代**，只能通过元素索引(或tie解包)进行获取元素的值。但是**给定的索引必须是在编译器就已经给定**，不能在运行期进行动态传递，否则将发生编译错误。

2. std::tie(): 解包时，我们如果只想解某个位置的值时，可以用std::ignore占位符来表示不解某个位置的值

   ```c++
   #include <iostream>
   #include <tuple>
   
   int main() {
       auto t1 = std::make_tuple(1, "PI", 3.14);
       auto t2 = std::make_tuple(2, "MAX", 999);
       std::cout << '(' << std::get<0>(t1) << ',' << std::get<1>(t1) <<
               ',' << std::get<2>(t1) <<  ')' << '\n';
       int num = 0;
       std::string name;
       double value;
       std::tie(num, name, value) = t1;
       std::cout << '(' << num << ',' << name << ',' << value << ')' << '\n';
       num = 0, name = "", value = 0;
       std::tie(std::ignore, name, value) = t2;
       std::cout << '(' << num << ',' << name << ',' << value << ')' << '\n';
       return 0;
   }
   /*
   output:
   (1,PI,3.14)
   (1,PI,3.14)
   (0,MAX,999)
   */
   ```

## 其他方法

1. 获取元素个数：

   可以采用`std::tuple_size<T>::value`来获得，其中T必须要显式给出tuple的类型；

   ```c++
   #include <iostream>
   #include <tuple>
   
   int main() {
       auto t1 = std::make_tuple(2, "MAX", 999, 888, 65.6);
       std::cout << "The t1 has elements: " << std::tuple_size<decltype(t1)>::value << '\n';
       return 0;
   }
   /*
   output: The t1 has elements: 5
   */
   
   ```

2. 获取元素类型：

   可以直接采用`std::tuple<size_t i,decltype(tuple)>::type`来获取；

   ```c++
   include <iostream>
   #include <tuple>
   
   int main() {
       auto t1 = std::make_tuple(2, "MAX", 999.9);
       std::cout << "The t1 has elements: " << std::tuple_size<decltype(t1)>::value << '\n';
       std::tuple_element<0, decltype(t1)>::type type0;
       std::tuple_element<1, decltype(t1)>::type type1;
       std::tuple_element<2, decltype(t1)>::type type2;
       type0 = std::get<0>(t1);
       type1 = std::get<1>(t1);
       type2 = std::get<2>(t1);
       std::cout << "type0 : " << type0 << '\n';
       std::cout << "type1 : " << type1 << '\n';
       std::cout << "type2 : " << type2 << '\n';
       return 0;
   }
   /*
   output:
   The t1 has elements: 3
   type0 : 2
   type1 : MAX
   type2 : 999.9
   */
   ```

3. 使用tuple引用来改变tuple内元素的值：

   ```c++
   int main() {
   	int a, b, c;
   	std::tuple<int&, int&> t = std::make_tuple(std::ref(a), std::ref(b));
   	
   	a = 1; b = 2;
   	
   	std::cout << std::get<0>(t) << " " << std::get<1>(t) << " " << std::endl;
   	
   	a = 2; b = 3;
   	std::cout << std::get<0>(t) << " " << std::get<1>(t) << " " << std::endl;
   }
   
   /*
   	输出：
   	1 2
   	2 3
   */
   ```

   

