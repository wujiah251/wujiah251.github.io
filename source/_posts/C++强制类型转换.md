title: C++强制类型转换
author: Jiahao Wu
tags: []
categories:
  - C++
date: 2021-02-16 20:35:00
---
C++提供四种强制类型转换方式：static_cast, dynamic_cast, const_cast, reinterpret_cast。  

# ``static_cast``的使用

``static_cast``可以用于：  
1. 基本类型之间的类型转换；  
2. void\*指针转化为其他类型指针；  
3. 添加const属性（``static_cast<const type&>(obj)``），强制转换的目标类型必须是指针或引用；  
4. 用于有继承关系对象之间的转换和类指针之间的转换：
 - 进行上行转换（把派生类的指针或引用转换成基类表示）是安全的；  
 - 进行下行转换（把基类的指针或引用转换为派生类表示），由于没有动态类型检查，所以是不安全的；

# ``const_cast``的使用

用于去除变量的只读属性；  
强制转换的目标类型必须是指针或引用；  

# ``dynamic_cast``的使用

1. 用于有继承关系的类指针间的转换；
2. 用于有交叉关系的类指针间的转换；
3. 具有类型检查的功能；
4. 需要虚函数的支持。