title: C和C++内存管理
author: Jiahao Wu
date: 2021-02-16 17:08:44
tags:
---
C语言中我们一般用``malloc()``函数和``free()``函数来分配和释放内存。C++中我们一般用关键字``new``、``delete``来分配和释放内存。那他们有哪些区别呢？

# 使用方法

```C++
	Test* p_c = (Test*)malloc(sizeof(Test));
   Test* p_cpp = new Test;
   free(p_c);
   delete p_cpp;
```

# 不同点

``malloc()``以字节为单位分配内存，不会调用构造函数，而``new``以具体类型为单位分配内存，调用了构造函数。  
``free()``仅归还内存，析构函数未被调用。``delete``销毁对象，归还内存，析构函数被调用。  

``malloc()``和``new``区别：  
- new 在所有 C++ 编译器中都被支持  
- malloc 在某些系统开发中不能调用  
- new 能够触发构造函数的调用  
- malloc 仅分配需要的内存空间  
- 对象的创建只能使用 new  
- malloc 不适合面向对象开发  

``free()``和``delete``区别：  
- delete 在所有 C++ 编译器中都被支持  
- free 在某些系统开发中不能调用  
- delete 能够触发析构函数的调用  
- free 仅归还之前分配的内存空间  
- 对象的销毁只能使用 delete  
- free 不适合面向对象开发
