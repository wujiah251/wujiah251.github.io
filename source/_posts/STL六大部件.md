title: STL六大部件
author: Jiahao Wu
tags:
  - STL
categories:
  - C++
date: 2021-02-28 18:02:00
---
STL有六大部件（Components）：  
- 容器（Containers）  
- 分配器（Allocators）  
- 算法（Algorithms）  
- 迭代器（Iterators）  
- 适配器（Adapters）  
- 仿函数（Functors）


**前闭后开区间**：  
标准库规定所有容器都提供``begin()``、``end()``两个迭代器，前一个指向第一个元素，后一个指向最后一个元素的下一个位置。  
