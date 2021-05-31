title: C++编译链接模型精要
author: 乡村程序员
tags:
  - C++
categories:
  - C++
date: 2021-05-22 21:38:00

---

```c++
void foo(int)
{
    printf("foo(int);\n");
}
void bar()
{
    foo('a');
}
void foo(char)
{
    printf("foo(char);\n");
}
int main()
{
    bar();
}
```

C++的函数重载决议，当C++编译器读到一个函数调用语句时，它必须从目前已看到的同名函数中选出最佳函数。哪怕后面的代码中出现了最合适的匹配。这意味着如果我们交换两个namespace级的函数定义在源代码中的位置，那么有可能改变程序的行为。

