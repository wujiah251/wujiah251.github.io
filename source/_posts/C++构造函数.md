title: C++构造函数
author: Jiahao Wu
tags: []
categories:
  - C++
date: 2021-02-24 21:59:00
---
C++的构造函数有四种，默认构造函数、参数构造函数、拷贝构造函数、转换构造函数

## 默认构造函数

没有参数，如下：
```C++
class Student
{
	Student(){
    num=100;
    age=18;
   }
};
```

## 参数构造函数

传入了一些参数，如下：
```C++
class Student
{
	Student(int Age,int Num){
   		num=Num;
      age=Age;
   }
};
```

## 拷贝构造函数

形式参数是本类对象的引用：
```C++
class Student
{
	Student(Student& s){
   		age=s.age;
      num=s.num;
   }
};
```

## 转换构造函数

转换构造函数能够将其他类型的变量转换为本类型，参数只有一个：

```C++
class Student{
	Student(int r){
   		num=r; 
   }
};
```
如果Student类重载了+号，则下面的加法会调用student(18);
```C++
Student s1(10,10);
Student s2(s1+18);
```