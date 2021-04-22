title: lab2
author: Jiahao Wu
tags: []
categories:
  - 操作系统
  - 操作系统课程
  - ''
date: 2021-03-15 09:43:00
---
# lab2
## makefile

1. 本程序的编译使用哪个编译器？  
本程序使用的编译器是``gcc``。  
2. 采用哪个命令，可以将所有程序全部编译？  
使用``make``或``make all``命令可以全部编译。  
3. 采用哪个命令，可以将所有上次编译的结果全部删除？
使用``make clean``可以讲编译结果全部删除。  
4. 文件中第几行生成 btest 的目标文件？
在文件的地10-11行：  
```makefile
btest: btest.c bits.c decl.c tests.c btest.h bits.h
	$(CC) $(CFLAGS) $(LIBS) -o btest bits.c btest.c decl.c tests.c
```
5. 文件中第几行生成 fshow 的目标文件？
在文件的第13-14行：  
```makefile
fshow: fshow.c
	$(CC) $(CFLAGS) -o fshow fshow.c
```
6. 如果在 Makefile 文件中用要引用变量“FOO“， 怎么表示？
只需要``$(FOO)``即可。  

## 位级运算、数的编码

### 运行结果

```Linux
wujiahao@ubuntu:~/Desktop/os/lab2/datalab$ ./btest
Score	Rating	Errors	Function
 2	2	0	allOddBits
 4	4	0	isLessOrEqual
 4	4	0	logicalNeg
 5	5	0	floatScale2
 5	5	0	floatFloat2Int
Total points: 20/20
```
5个函数：
```C++
int allOddBits(int x)
{
   int num = 0xAAAAAAAA;
   return !((num & x) ^ num);
}
int isLessOrEqual(int x, int y)
{
   int a = x >> 31;
   int b = y >> 31;
   return (!((!a) & b)) & ((a & !b) | !((y + (~x + 1)) & (1 << 31)));
}
int logicalNeg(int x)
{
   return ((~(~x + 1) & ~x) >> 31) & 1;
}
unsigned floatScale2(unsigned uf)
{
   unsigned frac = uf & 0x007fffff;
   unsigned exp = (uf >> 23) & 0xff;
   unsigned s = uf >> 31;
   if (!exp)
   {
      frac <<= 1;
      if (frac >> 23)
      {
         frac = frac & 0x007fffff;
         exp++;
      }
   }
   else if (exp != 0xff)
   {
      exp++;
   }

   return (s << 31) | (exp << 23) | frac;
}
int floatFloat2Int(unsigned uf)
{
   //unsigned frac = uf & 0x007fffff;
   int frac = (uf & 0x007fffff) | 0x00800000;
   int exp = ((uf >> 23) & 0xff) - 127;
   int s = uf >> 31;
   int tar = 0;
   if (s)
      s = -1;
   else
      s = 1;

   if (exp > 31)
      return 0x80000000;
   if (exp < 0)
      return 0;
   if (exp > 23)
      tar = s * (frac << (exp - 23));
   else if (exp < 23)
      tar = s * (frac >> (23 - exp));
   return tar;
}
```
