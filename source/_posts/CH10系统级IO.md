title: CH10系统级IO
author: Jiahao Wu
categories: 深入理解计算机系统
date: 2021-01-14 15:39:00
tags:
---
# 系统级I/O

## Unix I/O

所有的Linux文件就是一个m字节的序列，所有的I/O设备（例如网络、磁盘、和终端）都被模型化为文件，而所有的输入输出都被当作相应文件的读和写来执行。这种设备优雅地映射为文件的方式，允许Linux内核引出一个简单、低级的应用接口，称为Unix I/O，这使得所有的输入和输出都能以统一且一致的方式执行。  
- 打开文件（内核会返回一个非负整数，叫做描述符，内核记录有关这个打开文件的所有信息，应用程序只需要记住这个描述符）  
- Linux shell创建的每个进程开始时都有三个打开的文件：标准输入（描述符为0）、标准输出（描述符为1）和标准错误（描述符为2）。头文件``<stdlib.h>``定义了常量STDIN_FILENO、STDOUT_FILENO和STDERR_FILENO，它们可以来代替显式的描述符值。  
- 改变当前的文件位置：对于每个打开的文件，内核保持着一个文件位置k，这是从问及那开头其实的字节偏移量。  
- 读写文件；读n个字节：k增加到k+n，给定一个大小为m字节的文件，当k>=m时执行读操作会触发一个称为end-of-file（EOF）的条件。  
- 关闭文件

## 文件

- 普通文件：包含任意数据。应用程序通常要区分文本文件（text file）和二进制文件（binary file），文本文件时只含有ASCII或Unicode字符的普通文件。二进制文件是所有其他文件。对内核而言，都一样。  
- 目录：包含一组连接（link）的文件，其中每个链接都将一个文件名映射到一个文件，这个文件可能时另一个目录。每个目录至少包含两个条目：“.”时到该目录自身的条目，以及“..”时到目录层次结构中父目录的链接。可以用mkdir创建一个目录，用ls查看其内容，用rmdir删除该目录。  
- 套接字（socket）是用来与另一个进程进行跨网络通信的文件。  
其他文件类型包含命名通道、符号链接，以及字符和块设备，这些不在本书讨论范围内。  
Linux将所有文件组织成一个目录层次结构，有名为“/”的根目录确定。  
作为其上下文的一部分，每个进程都有一个当前工作目录来确定其在目录层次结构中的当前位置。你可以用cd来修改shell中的当前工作目录。

## 打开和关闭文件

```C
#include<sys/types.h>
#include<sys/stat.h>
#include<fcnt1.h>
int open(char *filename,int flags,mode_t mode);
```
open函数将filename转换成一个文件描述符，并返回描述符数字。flags指明了进程打算如何访问这个文件：  
- O_RDONLY：只读  
- O_WRONLY：只写  
- O_RDWR：可读可写   
mode参数指定了新文件的访问权限位。  
最后，进程通过调用close函数关闭一个打开的文件。

## 读和写文件

应用程序分别调用read和write函数来执行输入和输出的。
```C++
#include<iostream>
ssize_t read(int fd,void *buf,size_t n);
// 返回：若成功则为读的字节数，若EOF则为0，若出错为-1
ssize_t write(int fd,const void *buf,size_t, n)
// 返回：若成成功则为写的字节数，若出错则为-1
// ssize_t-long，size_t-unsigned long
```
在某些情况下，read和write传送的字节比应用程序要求的要少。这些不足值不表示有错误。出现这样情况的原因有：  
- 读时遇到EOF。  
- 从终端读文本行。如果打开文件是与终端相关联的（如键盘和显示器），那么每个read函数将一次传送一个文本行，返回的不足值等于文本行的大小。  
- 读和写网络套接词（socket）。如果打开文件对应于网络套接字，那么内部缓冲约束和较长的网络延迟会引起read和write返回不足值。  
实际上，除了EOF，在读磁盘文件时不会遇到不足值，而在写磁盘文件时也不会遇到不足值。然而如果想创建健壮的（可靠的）诸如Web服务器这样的网络应用，就必须通过反复调用read和write处理不足值，直到所有需要的字节都传送完毕。

## 用RIO（Robust I/O）包健壮地读写

- 无缓冲的输入输出函数。这些函数直接在内存和文件之间传送数据，没有应用级缓冲。它们对将二进制数据读写到网络和从网路读写二进制数据尤其有用。  
- 带缓冲的输入函数。这些函数允许高效地从文本读取文本行和二进制数据，这些文件的内容缓存在应用级缓冲区内，类似像printf这样标准I/O函数提供的缓冲区。  

### RIO的无缓冲的输入输出函数

通过调用rio_readn和rio_writen函数，应用程序可以在内存和文件之间直接传送数据。
```C++
#include "csapp.h"
ssize_t rio_readn(int fd,void *usrbuf,size_t n);
ssize_t rio_writen(int fd,void *usrbuf,size_t n);
ssize_t rio_readn(int fd,void *usrbuf,size_t n){
    size_t nleft=n;
    ssize_t nread;
    char *bufp=usrbuf;
    while(nleft>0){
        if((nread=read(fd,bufp,nleft))<0){
            if(errno==EINTR)//遇到中断
                nread=0;
            else
                return -1;
        }
        else if(nread==0)break;
        nleft-=nread;
        bufp+=nread;
    }
    return (n-nleft);
}
```

对于同一个文件描述符可以任意交错地调用rio_readn和rio_writen。

### RIO的带缓冲输入函数

```C++
#include "csapp.h"
void rio_readinitb(rio_t *rp,int fd);
ssize_t rio_readlineb(rio_t *rp,void *usrbuf,size_t maxlen);
ssize_t rio_readnb(rio_t *,void *usrbuf,size_t n);
```
每打开一个描述符，都会调用一次rio_readinitb函数。它将描述符fd和地址rp处在一个类型为rio_t的读缓冲区联系起来。  
对于同一描述符，rio_readlineb和rio_readnb的调用可以交叉进行。

## 读取文件元数据

应用程序能够调用stat和fstat函数，检索到关于文件的信息（有时也称为文件的元数据）。
```C++
#include<unistd.h>
#include<sys/stat.h>
int stat(const char *filename,struct stat *buf);
int fstat(int fd,struct stat *buf);
```
stat函数以一个文件名作为输入，并填写如下所示的一个stat数据结构中的各个成员。fstat函数是相似的，只不过是以文件描述符而不是文件名作为输入。当我们在11.5节讨论Web服务器时，会需要stat数据结构中的st_mode和st_size成员。
```C++
struct stat{
    dev_t   st_dev;
    ino_t   st_ino;
    mode_t  st_mode;
    nlink_t st_nlink;
    uid_t   st_uid;
    gid_t   st_gid;
    dev_t   st_rdev;
    off_t   st_size;
    unsigned long st_blksize;
    unsigned long st_blocks;
    time_t  st_atime;
    time_t  st_mtime;
    time_t  st_ctime;
}
```


## 读取目录内容

函数opendir和readdir，closedir用来关闭流并释放其所有资源。

## 共享

内核用三个相关的数据结构来表示打开的文件：
- 描述符表（每个进程都有独立的描述符表，每个描述符表项指向文件表中的一个表项）
- 文件表（打开文件的集合是由一张文件表来表示，所有进程共享这张表。每个文件表的表项组成包括当前的文件位置、引用计数，以及一个指向v-node表中对应表项的指针。关闭一个描述符会减少相应的文件表表项中的引用计数。内核不会删除这个文件表表项，直到它的引用次数为0。）
- v-node表。所有进程共享，每个表项包含stat结构中的大多数信息，包括st_mode和st_size成员。  
多个描述符也可以通过不同的文件表表项来引用同一个文件。例如，如果以同一个filename调用open函数两次，就会发生这种情况。

## I/O重定向

Linux shell提供了I/O重定向操作符，允许用户将磁盘文件和标准输入输出联系起来。例如，键入
``linux> ls > foo.txt``
使得shell加载和执行ls程序，将标准输出重定向到磁盘文件foo.txt。就如我们将在11.5节中看到的那样，当一个Web服务器代表客户端允许CGI程序时，它就执行一种类似类型的重定向。
一种重定位方式：
```C
#include<unistd.h>
int dup2(int oldfd,int newfd);
```
dup2函数复制描述符表项oldfd到描述符表项newfd，覆盖描述符表项newfd以前的内容。

## 标准I/O

标准I/O库将一个打开的文件模型化为一个流。对于程序员而言，一个流就是一个指向FILE类型的结构的指针。每个ANSI C程序开始时都有3个打开的流stdin、stdout和stderr，分别对应于标准输入、标准输出和标准错误。  
类型为FILE的流是对文件描述符和缓冲流的抽想。流缓冲区的目的和RIO读缓冲区一样：就是使开销较高的Linux I/O系统调用的数量尽可能得小。

## 综合：我们该使用哪些I/O函数