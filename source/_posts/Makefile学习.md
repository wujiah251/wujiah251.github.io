title: Makefile学习
author: Jiahao Wu
categories: Linux C/C++
date: 2021-01-23 20:27:27
tags:
---
# Linux编译过程


1. -E 预处理：把 .h、.c文件展开形成一个个文件。宏定义直接替换头文件、库文件。最后生成.i文件。
```shell
> gcc -E hello.c -o hello.i
```
2. -S 汇编： .i文件生成一个汇编代码文件 .s
```shell
> gcc -S hello.i -o hello.s
```
3. -c 编译： .s文件生成一个.o文件
```shell
> gcc -C hello.s -o hello.o
```
4. -o 链接：生成可运行文件 hello
```shell
> gcc hello.o hello
```
5. 也可以直接来
```shell
> gcc hello.c -o hello
```


# 脚本语言：Makefile


Linux C/C++ 必须要使用的一个编译脚本。
Makefile 大型项目开发。
1. 第一层：显示规则
```Makefile
#是注释
#第一层：显示规则
#目标文件：依赖文件
#[TAB]指令
#
#第一个目标文件应该是我的最终文件！！！（递归思维）
#
#rm -rf hello.o hello.S hello.i hello
#伪目标：.PHONY
hello:hello.o
	gcc hello.o -o hello
hello.o:hello.S
	gcc -c hello.S -o hello.o
hello.S:hello.i
	gcc -S hello.i -o hello.S
hello.i:hello.c
	gcc -E hello.c -o hello.i

.PHONY:
clearall:
	rm -rf hello.o hello.S hello.i hello
clear:
    rm -rf hello.o hello.S hello.i
```
2. 第二层
```Makefile
#变量 =(替换)  +=(追加)  :=(常量)
TAR = test
OBJ = circle.o cube.o main.o
CC := gcc

$(TAR):$(OBJ)
    $(CC) -o $(TAR)

circle.o:circle.c
	$(CC) -c circle.c -o circle.o
cube.o:cube.c
	$(CC) -c cube.c -o cube.o
main.o:main.c
	$(CC) -c main.c -o main.o
```
3. 第三层：隐含规则
```Makefile
# %.c $.o 任意的.c或者.o   *.c *.o所有的.c .o
TAR = test
OBJ = circle.o cube.o main.o
CC := gcc

$(TAR):$(OBJ)
	$(CC) $(OBJ) -o $(TAR)

%.o:%.c
	$(CC) -c %.c -o %.o 
```
4. 第四层：通配符
```Makefile
# $@所有的目标文件
# $^所有的依赖文件
# $<所有的依赖文件的第一个文件
TAR = test
OBJ = circle.o cube.o main.o
CC := gcc
RMRF := rm -rf

$(TAR):$(OBJ)
	$(CC) $^ -o $@

%.o:%.c
	$(CC) -c $^ -o $@ 
.PHONY:
clearall:
    $(RMRF) $(OBJ) $(TAR)
clear:
    $(RMRF) $(OBJ)
```