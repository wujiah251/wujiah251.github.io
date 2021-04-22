title: lab1
author: Jiahao Wu
tags: []
categories:
  - 操作系统
  - 操作系统课程
date: 2021-03-06 00:13:00
---
# Lab1实验报告

## 练习1 gcc

1. 编译
```Linux
$ gcc -o glory glory.c
// gcc表示使用C编译器（c++是g++），-o参数表示生成可执行文件为glory，glory.c是原文件
```
2. 运行
```Linux
$ ./glory
// 运行可执行文件file就是：./file
```
3. 修改glory.c宏定义如下：
```C++
#define V0 3
#define V1 3
#define V2 3
#define V3 1
```
重新编译并允许结果如下：
```
  SJTU labstarter:
====================
Happy Happy Happy 
FPX

Guo Li!
```


## 练习2 GDB

1. How do you pass command line arguments to a program when using gdb?  
修改hello.c的主函数如下：  
```C++
int main(int argc, char** argv) {
        int i;
        if(argc>1){
            printf("%s\n",argv[1]);
        }
        char *str ="hello, world!", ch;
        for (i = 0; i < strlen(str); i++)
                ch = str[i];

        printf("%s\n",str);

        return 0;
}
```
编译并调试如下：
```
$ gcc -g -o hello hello.c
$ gdb hello
...
(gdb) run
Starting program: /home/wujiahao/Desktop/os/lab1/hello 
hello, world!
[Inferior 1 (process 6122) exited normally]
(gdb) run arg_test
Starting program: /home/wujiahao/Desktop/os/lab1/hello arg_test
arg_test
hello, world!
[Inferior 1 (process 6128) exited normally]
```
``run``直接允许程序，由于没有命令行参数所以直接输出了``hello,wolrd!``。在run后面添加参数可以直接输出结果，比如``run arg_test``结果多输出了一行``arg_test``。  
2. How do you set a breakpoint which only occurs when a set of conditions is true (e.g. when certain variables are a certain value)?  
想设置一个条件断点，可以利用break if命令，如下所示：
```Linux
(gdb) break line-or-function if expr
```
例如：
```Linux
(gdb) break 46 if testsize==100
```
我面这样修改hello.c文件：
```C++
int main(int argc, char** argv) {
        int i;
        char test_num='\0';
        if(argc>1){
            test_num=argv[0][0];
        }
        char *str ="hello, world!", ch;
        for (i = 0; i < strlen(str); i++)
                ch = str[i];

        printf("%s\n",str);

        return 0;
}
```
```Linux
(gdb) break 18 if test_num!='\0'
Breakpoint 1 at 0x4005cc: file hello.c, line 18.
(gdb) run
Starting program: /home/wujiahao/Desktop/os/lab1/hello 
hello, world!
[Inferior 1 (process 6217) exited normally]
(gdb) run 1
Starting program: /home/wujiahao/Desktop/os/lab1/hello 1
Breakpoint 1, main (argc=2, argv=0x7fffffffde08) at hello.c:18
18		printf("%s\n",str);
```

我们在18行设置一个条件断点，中断条件为字符变量test_num非'\0'空字符，我们可以通过命令行参数修改这个字符来触发中断。可以发现当时有命令行参数1的情况下，这个程序会中断在18行（就是``printf``这行），不会输出``hello,world!``。  
3. How do you execute the next line of C code in the program after stopping at a breakpoint?  
使用命令``continue``可以接着执行程序直到终止或者再次遇到断点，命令``step``可以运行下一行代码。  
4. If the next line of code is a function call, you'll execute the whole function call at once if you use your answer to #3. How do you tell GDB that you want to debug the code inside the function instead?  
``step``可以只执行下一行代码，不会调用整个函数，也可以通过``break [file:]function``在函数中设置断点。  
5. How do you resume the program after stopping at a breakpoint?  
运行``continue``即可恢复程序执行。  
6. How can you see the value of a variable (or even an expression like 1+2) in gdb?  
``print``可以查看变量，比如对程序hello调用``print str``：  

```
(gdb) break 18
Breakpoint 1 at 0x4005cc: file hello.c, line 18.
(gdb) run
Starting program: /home/wujiahao/Desktop/os/lab1/hello 

Breakpoint 1, main (argc=1, argv=0x7fffffffde08) at hello.c:18
18		printf("%s\n",str);
(gdb) print str
$1 = 0x400674 "hello, world!"
(gdb) print 1+2
$2 = 3
```

输出了字符指针str的值，以及指向字符串的值。  
7. How do you configure gdb so it prints the value of a variable after every step?
通过``display``命令可以在程序中断时候自动显示变量的值，比如``display ch``可以自动显示字符变量ch的值。  
8. How do you print a list of all variables and their values in the current function?  
``bt``可以显示程序堆栈信息，然后``bt full``可以查看当前函数的所有变量。  
9.  How do you exit out of gdb?  
``quit``可以退出gdb调试。  

## 练习3 调试

编译并执行
```Linux
$ gcc -g -o ll_equal ll_equal.c
$ ./ll_equal
```
结果如下：
```
equal test 1 result = 1
Segmentation fault (core dumped)
```
然后我们开始调试
```C++
(gdb) break 10
Breakpoint 2 at 0x4005a2: file ll_equal.c, line 10.
(gdb) continue
Continuing.

Breakpoint 2, ll_equal (a=0x7fffffffdc70, b=0x7fffffffdc70) at ll_equal.c:10
10		while (a != NULL) {
(gdb) continue
Continuing.
equal test 1 result = 1

Breakpoint 2, ll_equal (a=0x7fffffffdc70, b=0x7fffffffdc90) at ll_equal.c:10
10		while (a != NULL) {
(gdb) continue
Continuing.

Program received signal SIGSEGV, Segmentation fault.
0x00000000004005ae in ll_equal (a=0x7fffffffdc90, b=0x0) at ll_equal.c:11
11			if (a->val != b->val)
```
最后发现在11行出现错误，b的值为``0x0``，不能对空指针取调用``->``运算符。修改ll_equal.c如下：
```C++
#include <stdio.h>

typedef struct node {
        int val;
        struct node* next;
} node;

/* FIXME: this function is buggy. */
int ll_equal(const node* a, const node* b) {
        while (a != NULL&&b != NULL) {
                if (a->val != b->val)
                        return 0;
                a = a->next;
                b = b->next;
        }
        /* lists are equal if a and b are both null */
        return a == b;
}

int main(int argc, char** argv) {
        int i;
        node nodes[10];

        for (i=0; i<10; i++) {
                nodes[i].val = 0;
                nodes[i].next = NULL;
        }

        nodes[0].next = &nodes[1];
        nodes[1].next = &nodes[2];
        nodes[2].next = &nodes[3];

        printf("equal test 1 result = %d\n", ll_equal(&nodes[0], &nodes[0]));
        printf("equal test 2 result = %d\n", ll_equal(&nodes[0], &nodes[2]));

        return 0;
}
```
重新编译运行结果如下：
```
equal test 1 result = 1
equal test 2 result = 0
```


## 练习4 Make初步


```Linux
wujiahao@ubuntu:~/Desktop/os/lab1$ make wc
cc     wc.c   -o wc
wujiahao@ubuntu:~/Desktop/os/lab1$ ./wc wc.c
wujiahao@ubuntu:~/Desktop/os/lab1$ wc wc.c
  9  23 145 wc.c
wujiahao@ubuntu:~/Desktop/os/lab1$ which wc
/usr/bin/wc
```
前一次是调用我们生成的可执行文件wc，源文件wc.c的文件名作为参数。后者是调用系统的命令wc，参数是wc.c，这个wc是系统自己的可执行文件，位于/usr/bin/wc。

修改代码如下：
```C++
#include <stdio.h>
#include <stdlib.h>
#include <ctype.h>

void wc(FILE *ofile, FILE *infile) {
    char ch, pre_ch='\0';
    int lines=0,words=0,chars=0;
    while((ch=fgetc(infile))!=EOF){
        ++chars;
        if(ch!=' '&&ch!='\n'&&ch!='\t'){
            if(pre_ch==' '||pre_ch=='\0'||pre_ch=='\t'||pre_ch=='\n')++words;
        }
        else if(ch=='\n')++lines;
        pre_ch=ch;
    }
    fprintf(ofile,"\t%d\t%d\t%d\n",lines,words,chars);
}
int main (int argc, char *argv[]) {
    char* filename;
    FILE *infile=stdin;
    if(argc>1){
        filename=argv[1];
        infile=fopen(filename,"r");
    }
    wc(stdout,infile);
    fclose(infile);
    return 0;
}
```

结果如下：
```Linux
wujiahao@ubuntu:~/Desktop/os/lab1$ ./wc wc.c
	29	49	616
wujiahao@ubuntu:~/Desktop/os/lab1$ wc wc.c
 29  49 616 wc.c
wujiahao@ubuntu:~/Desktop/os/lab1$ wc
123123 123123
1231
123
123123 123123 123123
123 1231 
      5       9      53
wujiahao@ubuntu:~/Desktop/os/lab1$ ./wc
123123 123123
1231
123
123123 123123 123123
123 1231
	5	9	53
```