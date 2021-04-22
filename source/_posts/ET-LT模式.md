title: I/O多路复用
author: Jiahao Wu
date: 2021-02-05 21:13:50
tags:
---
I/O多路复用是一种机制：一个进程监听多个描述符，一旦一个某个描述符就绪（读就绪或者写就绪），通知进程进行相应的读写操作。  
这个机制在Linux中有3个常见实现（select、poll、epoll），但是他们本质都是同步I/O，因为他们都需要在读写事件就绪后自己负责读写。而异步I/O不需要负责读写，异步I/O的实现会负责将数据从内核空间拷贝到内核空间。  

## Select机制

```C++
int select (int n, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout); 
void FD_ZERO(fd_set *fdset);	//清空一个fdset
void FD_SET(int fd, fd_set *fdset);	//添加一个fd进入fdset
void FD_CTR(int fd,fd_set *fdset);	//从fdset删除一个fd
int FD_ISSET(int fd, fd_set *fdset);	//判断fdset里面是否有某个fd
```
select函数监听的文件描述符分3类，分别是writefds、readfds和exceptfds。调用后select函数会阻塞，知道有描述符就绪，或者超时，函数返回。当select函数返回后，可以通过遍历fdset来找到就绪的描述符。  
select目前在几乎所有平台上支持，其良好的跨平台支持也是它的一个优点。缺点就是单个进程能够监听的描述符的数量存在最大限制，在Linux上一般为1024，可以通过修改宏定义来提升这一限制，但是会降低效率。


## Poll机制
```C++
int poll (struct pollfd *fds, unsigned int nfds, int timeout); 
```
不同于select使用3个位图来表示3个fdset的方式，poll使用一个pollfd的指针实现。
```C++
struct pollfd{
	int fd;
	short events;
   short revents;
}
```
pollfd结构包含了要监视的event和发生的event，不再使用select“参数-值”传递方式。同时，pollfd并没有最大数量限制（但是数量大了性能变差）。和select一样，poll返回后需要轮询pollfd来获取就绪的描述符。

从上面的例子看，select和poll都需要在返回后同遍历文件描述符来获取已经就绪的socket。事实上，同时连接的大量客户端在一时刻可能只有很少的处于就绪状态，因此随着监听的描述符数量增长，其效率线性下降。

## Epoll

epoll是select和poll的增强版本。epoll更加灵活，没有数量限制。epoll使用一个文件描述符管理多个描述符，将用户关系的文件描述符的事件存放在内核的一个事件表中，这样在用户空间和内核空间的copy只需一次。  

### Epoll具体使用

epoll操作过程需要3个接口，分别如下所示：
```C++
int epoll_create(int size);
int epoll_ctl(int epfd, int op, struct epoll_event *event);
int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout);
```


- ``epoll_create()``创建一个epoll句柄，size用来告诉内核这个监听的数目一共多大，这个参数无法限制epoll监听的最大数目，而是给出一个建议。 epoll句柄会占用一个fd值。 
- ``epoll_ctl()``对指定的描述符fd执行op操作，可以添加（EPOLL_CTL_ADD）/删除（EPOLL_CTL_DEL）/修改（EPOLL_CTL_MOD）对fd的监听事件。event是告诉我们内核需要监听数目事件。``struct epoll_event``结构如下所示：

```C++
struct epoll_event{
	_uint32_t events;	/* Epoll events */
   epoll_data_t data;  /* User data variable */
};

//events可以表示为以下几个宏的集合
EPOLLIN ：表示对应的文件描述符可以读（包括对端SOCKET正常关闭）；
EPOLLOUT：表示对应的文件描述符可以写；
EPOLLPRI：表示对应的文件描述符有紧急的数据可读（这里应该表示有带外数据到来）；
EPOLLERR：表示对应的文件描述符发生错误；
EPOLLHUP：表示对应的文件描述符被挂断；
EPOLLET： 将EPOLL设为边缘触发(Edge Triggered)模式，这是相对于水平触发(Level Triggered)来说的。
EPOLLONESHOT：只监听一次事件，当监听完这次事件之后，如果还需要继续监听这个socket的话，需要再次把这个socket加入到EPOLL队列里 
```

- ``epoll_wait()``等待epfd上的I/O事件，最多返回maxevents个事件。参数events用来从内核得到事件的集合，maxevents告知内核这个events有多大。

### Epoll底层机制

Epoll在内核维护了两个数据结构，一个红黑树用来存储监听IO事件的fd，另一个用来存储IO事件的双链表。红黑树能够提供高效的fd查找、修改、删除操作（``epoll_ctl()``）。一旦某个fd发生了IO事件，就会触发fd的回调函数，将事件添加到双链表中。而``epoll_wait()``只需要检查链表是否为空即可，不为空等待返回，否则等待IO事件发生触发回调函数。  


## 工作模式

epoll对文件描述符有两种工作模式：**LT（level trigger，水平触发）和ET（edge trigger，边沿触发）**：LT模式是默认模式，LT模式与ET模式区别如下所示：  
**LT**：当epoll检测到描述符事件发生并将此事件通知主程序，应用程序可以不立即处理该事件。下次调用epoll_wait时，会再次响应应用程序并通知此事件。  
**ET模式**：当epoll_wait()检测到描述符事件发生并将此事件通知注应用程序，应用程序必须立即处理该事件。如果不处理，下次调用epoll_wait()时，不会再次响应应用程序并通知此事件。  

### LT模式

LT是epoll默认的工作方式，支持阻塞和非阻塞两种机制。LT模式下内核会持续通知你文件描述符就绪了，然后你可以对这个就绪的fd进行I/O操作。如果不做任何操作，内核还是会继续通知你的。  

### ET模式


ET模式相对LT模式更加高效，只支持非阻塞模式。在这个模式下，当描述符从未就绪变为就绪时，内核通过epoll告诉你。然后它会假设你知道文件描述符已经就绪，并且不再为那个文件描述符发生更多的就绪通知。直到你做了某些操作导致那个文件描述符不再为就绪状态了。  
ET模式在很大程度上减少了epoll事件被重复触发的次数，因此效率要比LT模式高。epoll工作在ET模式时，必须使用非阻塞套接口，以避免一个文件句柄的阻塞导致把其他文件描述符饿死。  

## Epoll总结

在select/poll中，进程只有在调用一定的方法后才对所有监视的文件描述符进行扫描，而epoll事先通过epoll_ctl()注册一个文件描述符，一旦某个文件描述符就绪时，内核会采取回调机制，迅速激活这个文件描述符，当进程调用epoll_wait()时便得到通知。  

epoll的优点主要是以下几个方面：  
1. 监听的文件描述符不受限制，它所支持的fd上限是最大可打开文件数目；  
2. I/O效率不会随着监听的文件描述符规模增大而下降。不同于select/poll轮询的方式，epoll通过每个fd定义的回调函数来实现。只有就绪的fd才会执行回调函数。