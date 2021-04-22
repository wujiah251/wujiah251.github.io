title: 迭代器概念与traits编程技法
author: Jiahao Wu
tags: []
categories:
  - C++
  - STL标准库与泛型编程
date: 2021-03-12 10:20:00
---
# 迭代器（iterators）概念与traits编程技法

## 迭代器设计思维-STL关键所在

STL的中心思想是讲数据容器（containers）和算法（algorithms）分开，最后再以一贴胶着剂将它们撮合在一起。前两者可以通过C++的class templates和function templates来实现，后者如何实现呢。

## 迭代器（iterators）是一种smart pointer

## 迭代器相应型别

可以利用function template来做类型推导：
```C++
template<class T, class T>
void func_impl(I iter, T t)
{
	T tmp;
   //...
};
template<class T>
inline
void func(I iter)
{
	func_impl(iter, *iter);
}

int main()
{
	int i;
	func(&i);
}
```

## Traits编程技法-STL源码门钥

返回值类型如何推导呢？可以使用内嵌类型：  
```C++
template<class T>
struct MyIter{
	typedef T value_type;
   T* ptr;
   MyIter(T* p=0):ptr(p){}
   T& operator*()const{return *ptr;}
   // ...
};

template<class T>
typename I::value_type
func(I ite)
{return *ite;}

// ...
MyIter<int> ite(new int(8));
cout << func(ite);	//输出：8
```

``typename``可以告诉编译器返回值是一个类型。但是这里有个问题，并不是所有的迭代器都是class type，原生指针就不是。  

下面这个``class template``专门用于“萃取”迭代器的特性，而value_type正是迭代器的特性之一：  
```C++
template<class I>
struct iterator_traits{
	typedef typename I::value_type value_type;
};
```
我们就可以通过如下方式萃取出I中的value_type了:  
```C++
template<class I>
typename iterator_traits<I>::value
func(I ite)
{return *ite;;}
```
还可以萃取原生指针中的value_type：  
```C++
template<class T>
struct iterator_traits<T*>{
	typedef T value_type;
}
```

根据经验，最常用到的迭代器相应类型有五种：``value type``,``difference type``,``pointer``,``reference``,``iterator catagoly``。  为了与STL能够水乳交融，一定要定义如下萃取机：  
```C++
tempalte<class I>
struct iterator_traits{
	typedef typename I::iterator_category    iterator_category;
   typedef typename I::value_type 		   value_type;
   typedef typename I::difference_type 	  difference_type;
   typedef typename I::pointer 			  pointer;
   typedef typename I::reference		     reference
};
```


















