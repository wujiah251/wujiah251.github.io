title: C++继承
author: Jiahao Wu
tags: []
categories:
  - C++
date: 2021-02-16 15:33:00
---
考虑继承关系时，我们涉及三种成员，分别是私有成员、公有成员、保护成员，关键字依次是``private``、``public``、``protected``。

保护成员在类的用户眼中，等价于私有成员，但是保护成员可以被类成员函数调用。  


# 公有继承

```C++
class Derive:public Base
//public表示公有继承
```

基类的私有成员-->派生类不可见；  
基类的公有成员-->派生类的公有成员；  
基类的保护成员-->派生类的保护成员。  

# 私有继承  

```C++
class Derive:private Base
//private表示私有继承
//私有继承也是默认的继承方式
class Derive:Base
```

基类的私有成员-->派生类不可见；  
基类的公有成员-->派生类的私有成员；  
基类的保护成员-->派生类的私有成员。 

# 保护继承

```C++
class Derive:protected Base
//protected表示保护继承
```
基类的私有成员-->派生类不可见；  
基类的公有成员-->派生类的保护成员；  
基类的保护成员-->派生类的保护成员。 





