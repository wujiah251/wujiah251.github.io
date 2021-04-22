title: 轻量级高性能Web服务器开发
author: Jiahao Wu
categories: Linux C/C++
date: 2021-02-12 21:26:43
tags:
---
# 起因


笔者这学期先后学习了《深入理解计算机系统》、《计算机网络：自顶向下方法》、《操作系统概念》，想要做一个项目来巩固这段时间所学的知识，考虑到CSAPP中实现了一个TinyWebServer，我打算开发一个高性能Web服务器。  


# 相关环境配置


- 服务器测试环境  
 - Ubuntu版本16.04
 - MySQL版本5.7.32
- 浏览器测试
 - FireFox
- 创建数据库  

```mysql
create database webserver_database;
// 创建一个名为webserver_database的数据库
user database webserver_database;
// 在这个数据库中创建表user，关系为(username,passwd)
// 保存用户名-密码
create table user(
	username char(50) NULL,
   passwd char(50) NULL
);
// 添加一条数据，内容可以填上任意的用户名和密码
INSERT INTO user(username, passwd) VALUES('wujiahao','wujiahao');
```  

这个数据库只有一个表user，其保存的是用户的用户名、密码。  
- 文件管理  
 - makefile  
 - 运行方式  
 
 ```shell
 linux> make
 linux> ./server
 ```


# 知识基础  


本项目中主要涉及以下知识。  
- C++基础  
- 数据结构基础  
- 计算机网络基础  
- 操作系统基础  
- Linux基础  
- 服务器开发规范  

项目中参考的书籍：  
- 《深入理解计算机系统》  
- 《计算机网络：自顶向下方法》  
- 《UNIX环境高级编程》  
- 《UNIX网络编程 卷一：套接字联网API》  
- 《鸟哥的Linux私房菜：基础篇》  
- 《Linux高性能服务器编程》  


## 计算机系统  


CSAPP主要是理解进程、线程的相关概念，对多进程、多线程编程有一个基本的认识。其中涉及到的进程、线程、信号量、互斥锁、生产者-消费者模型等概念在本项目中用到最多。重点参考本书的第三部分（第10章系统级I/O、第11章网络编程、第12章并发编程）。另外可以读一读第7章链接和第8章异常控制流，对项目文件的管理和理解用户态/内核态有帮助。 


## 计算机网络


《计算机网络：自顶向下方法》本书我实际上只看了前5章，包括应用层、传输层、网络层，由于计算机网络是一个层次结构，所以只需要掌握这部分内容，在进行服务器开发时，就不存在较大的知识盲区了。


## UNIX编程


《UNIX环境高级编程》、《UNIX网络编程 卷一：套接字联网API》主要是作为字典，遇到需要用的函数或者不理解的知识时进行查询用的。


## Linux基础


《鸟哥的Linux私房菜：基础篇》比较好的Linux入门书籍，由于笔者时间紧迫，就没有先系统学习了，只在需要的时候查询相关命令。


## 服务器开发规范


主要参考了游双的《Linux高性能服务器编程》，这本书提供了很多服务器开发中运用的技术，并提供了相应的代码（笔者在实际开发中很多代码是基于本书提供的代码所写）：  

- 主流服务器模型：C/S模型和P2P模型  
 - C/S模型  
也就是客户端/服务器模型，这个模型很简单，就是所有的资源都被服务器所拥有，客户端需要访问服务器来获取所需的资源。  
 - P2P模型  
 P2P模型比C/S模型更接近网络通信的实际情况，摒弃了以服务器为中心的格局，让网络中所有主机重新回到对等的地位。  
 - 比较  
C/S模型适合资源相对集中的场合，它的实现也更为简单，缺点是当访问量过大的时候，可能所有客户都将得到很慢的响应。P2P模型使得每台机器在消耗服务的同时也给别人提供服务，这样资源能够充分、自由地共享。但是当用户之间传输的请求过多时，网络的负载将会加重。  
- I/O模型：同步I/O与异步I/O  
 - 阻塞I/O  
 当发生系统调用时候，比如recv，用户进程要一直等待到数据拷贝到用户空间，这段时间内进程始终阻塞。它是同步I/O。  
 - 非阻塞I/O  
 还是以recv为例子，不管有没有数据都返回，如果没有数据，过一会再调用recv看看，如此循环。但是如果有数据，它要等待数据拷贝到用户空间，因此还是同步I/O。  
 - I/O复用  
 同时监听多个文件描述符，当有可读/可写/异常事件发生的时候，通知进程，进行读写或者处理异常。当没有事件发生时候，正常运行。就当前例子来看还是同步I/O，因为依旧要等待数据读完或者写完才能继续运行。I/O复用可用于同步I/O也可用于异步I/O。  
 - 信号驱动的I/O模型  
通过调用sigaction注册信号函数，等内核数据准备好的时候，系统中断当前程序，执行信号函数（在这里调用recv）。这还是同步I/O。  
 - 异步I/O模型  
 调用aio_read，让内核等数据准本好，并且复制到用户进程空间后执行实现指定好的函数。这样子省去了进程等待读数据的事件。这是真正的异步I/O。  
 - I/O的两个阶段  
  内核数据的准备阶段和内核数据复制到用户缓冲区阶段。只有进程不用等待这两个阶段，才是真正的异步I/O。  
- 事件处理模式：Reactor模式/Proactor模式    
服务器程序通常要处理3类事件，I/O事件、信号和定时事件。在处理事件上有两种高效的事件处理模式：Reactor和Proactor。这两种其实也都是I/O复用模型  
 - Reactor模式  
 Reactor是这样一种模式，它要求主线程只负责监听文件描述上是否有事件发生，有的话就立即将该事件通知工作线程。除此之外，主线程不做任何其他实质性的工作，读写数据、接收新的连接，以及处理客户请求均在工作线程中完成。  
 使用同步I/O模型（以``epoll_wait()``为例）实现的Reacotr模式的工作流程如下：  
 1）主线程往epoll内核事件表中注册socket上的读就绪事件。  
 2）主线程调用``epoll_wait()``等待socket上有数据可读。  
 3）当socket上有数据可读时，``epoll_wait()``通知主线程。主线程将socket可读事件放入请求队列。  
 4）睡眠在请求队列上的某个工作线程被唤醒，它从socket读取数据，并处理客户请求，然后往epoll内核事件表中注册该socket上的写事件。  
 5）主线程调用``epoll_wait()``等待socket可写。  
 6）当socket可写时，``epoll_wait()``通知主线程。主线程将socket可写事件放入请求队列。  
 7）睡眠在请求队列上的某个工作线程被唤醒，它往socket上写入服务器处理客户请求的结果。  
 重点：要有一个read/write事件处理器。  

- Proactor模式  
与Reactor模式不通，Proactor模式将所有I/O操作都交给主线程和内核来处理，工作线程仅仅负责业务逻辑。  
使用异步I/O模型（``aio_read``和``aio_write``为例）实现得Proactor模式工作流程。  
1）主线程调用aio_read函数向内核注册socket上的读完成事件，并告诉内核用户读缓冲区的位置，以及读操作完成时如何通知应用程序。（信号）  
2）主线程继续处理其他逻辑。  
3）应用程序预先定义好的信号处理函数选择一个工作线程来处理客户请求。工作线程处理完客户请求之后，调用``aio_write``函数向内核注册socket上的写完成事件，并告诉内核用户缓冲区的位置，以及写操作完成时如何通知应用程序（信号）。  
4）主线程继续处理其他逻辑。  
5）当用户缓冲区的数据被写入socket之后，内核将向应用程序发送一个信号，以通知应用程序数据已经发送完毕。  
6）当用户缓冲区的数据写入socket之后，内核向应用程序发送一个信号，以通知应用程序数据已经传送完毕。  
7）应用程序预先定义好的信号处理函数选择一个工作线程来做善后处理，比如决定是否关闭socket。  
- 同步模拟Proactor模型  
《Linux高性能服务器编程》一书中提到了一种用同步I/O来模拟Proactor模式的方法。其原理是：主线程执行数据读写操作，读写完成之后，主线程向工作线程通知这一“完成事件”。那么工作线程直接获得了读写结果，接下来只要做的只是对读写结果的处理。  
1）主线程调用``epoll_wait()``内核事件表中注册socket上的读就绪事件。  
2）主线程调用``epoll_wait()``等大socket上有数据可读。  
3）当socket上有数据可读时，``epoll_wait()``通知主线程。主线程从socket循环读取数据，直到没有更多数据可读，然后将所有数据封装成一个请求对象并插入请求队列。  
4）睡眠在请求队列中的某个工作流程被唤醒，它获得请求对象并处理客户请求，然后往epoll内核事件表中注册socket上的写就绪事件。  
5）主线程调用``epoll_wait()``等待socket可写。  
6）当socket可写时，``epoll_wait()``通知主线程。主线程往socket上写入服务器处理客户请求的结果。  
这里同步模拟和异步模拟Proactor的区别在于，同步模拟是主线程自己进行I/O操作，需要等待读完数据，而异步I/O是通过``aio_read()``异步读取，主线程不需要等待I/O完成。  
- 总结  
Reactor和Proactor的本质区别在于谁负责I/O操作，其实它们都可以分别利用同步I/O或者异步I/O来实现。
- 并发模式  
并发编程的目的是让程序“同时”执行多个任务，如果程序是计算密集型的，并发编程并没有优势，反而由于任务切换使得效率低。但如果程序是I/O密集型，则情况不通。和IO不通的是，在并发模式中，同步指的是程序完全按照代码序列的顺序执行；异步指的是程序的执行需要由系统事件来驱动。同步线程效率低，实时性差，但是逻辑简单，异步线程实时性强，效率高，但是相对复杂难以调试。对于服务器而言，既追求实时性，也要同时处理多个请求，我们应该采用半同步半异步模式。  
 - 半同步/半异步模式  
 同步线程处理客户逻辑，异步线程处理I/O事件。  
 - 领导者/追随者模式  
 多个工作线程轮流获得事件源，轮流监听、分发并处理事件。在任意时刻，程序都仅有一个领导者线程，它负责监听I/O事件。而其他线程则是追随者，它们休眠在线程池，然后等待处理I/O事件。此时新的领导者等待I/O事件，原来的领导者处理I/O事件，二者实现了并发。  
 
以上就是《Linux高性能服务器编程》中关于服务器开发的一些重要技术和规范。  


# 框架设计




# 线程安全


文件``lock/locker.h``提供了3类，分别是sem、locker、cond，是对信号量、互斥锁、条件变量的封装。  

## sem类
信号量，提供函数如下：
```C++
sem();          //默认构造函数
sem(int num);   //构造函数，信号量初始为num
~sem(); //析构函数
bool wait();    //V操作，信号量-1
bool post();    //P操作，信号+1
```


## locker类


互斥锁，提供函数如下：  
```C++
locker();   //构造函数
~locker()； //析构函数
lock()；    //上锁
unlock()；  //解锁
pthread_mutex_t *get()；    //返回互斥锁指针
```


## cond类


条件变量，提供的函数如下：
```C++
cond();//构造函数
~cond();//析构函数
bool wait(pthread_mutex_t *m_mutex);//等待目标条件变量
bool timewait(pthread_mutex_t *m_mutex, struct timespec t);//等待目标条件变量一定时间
bool signal();//信号唤醒
bool broadcast();//信号唤醒
```
``wait(pthread_mutex_t *m_mutex)``内部调用``pthread_cond_wait()``，此函数内部操作分为以下步骤：
- 先传入一个互斥锁和条件变量，将线程放在条件变量的请求队列后，内部解锁
- 线程等待被pthread_cond_broadcast信号唤醒或者pthread_cond_signal信号唤醒，唤醒后去竞争锁，若竞争到锁，则内部再次加锁。
使用方式举例说明，下面这是我们设计的用来处理日志事件的循环队列，利用上述三个类，这个队列实现了线程安全。条件变量这样使用：
```C++
bool push(const T &item)
{
    m_mutex.lock();
    if (m_size >= m_max_size)
    {
        m_cond.broadcast();
        m_mutex.unlock();
        return false;
    }
    m_back = (m_back + 1) % m_max_size;
    m_array[m_back] = item;
    m_size++;
    m_cond.broadcast();
    m_mutex.unlock();
    return true;
}
//pop时,如果当前队列没有元素,将会等待条件变量
bool pop(T &item)
{
    m_mutex.lock();
    while (m_size <= 0)
    {
        if (!m_cond.wait(m_mutex.get()))
        {
            m_mutex.unlock();
            return false;
        }
    }
    m_front = (m_front + 1) % m_max_size;
    item = m_array[m_front];
    m_size--;
    m_mutex.unlock();
    return true;
}
```
将事件``push``进队列之前现对互斥锁``m_mutex``上锁，然后添加成功后，解锁，并广播``m_cond.wait()``，因为``pop``可能因为队列为空而被阻塞，广播将唤醒等待队列中的条件变量。  
``pop``中先将``m_mutex``上锁，然后``m_cond_wait()``函数将条件变量添加到等待队列中后，将mutex_解锁（这样条件变量不会错过条件判断和加入等待队列之间的所有变化）。然后当条件变量满足时，将会去竞争互斥锁，然后竞争成功的条件变量加锁并返回（因为会有多个条件变量被满足来竞争互斥锁）。  


# 日志


在``log/block_queue.h``中实现了一个线程安全的循环队列，其功能和一般的队列区别不大。这个队列用于处理日志事件。  
在``log/log.h,log/log.cc``中声明并实现了日志类。


## block_queue类


一个线程安全的循环队列，操作和普通队列基本一致，不做过多说明。


## log类


日志类，采用单例模式。  
单例模式是最常用的设计模式之一，保证一个类只有一个实例，并提供一个访问它的全局访问点，该实例被所有程序模块共享。  
实现思路：私有化它的构造函数，以防止外界创建单例类的对象；使用类的私有静态指针变量窒息那个类的唯一实例，并用一个公有的静态方法获取该实例。  
单例模式有两种实现方法，分别是懒汉和饿汉模式。顾名思义，懒汉模式，即非常懒，不用的时候不去初始化，所以在第一次被使用时才进行初始化；饿汉模式，即迫不及待，在程序运行时立即初始化。
我的实现是这样的：
```C++
class Log
{
public:
    //C++11以后,使用局部变量懒汉不用加锁
    static Log *get_instance()
    {
        static Log instance;
        return &instance;
    }
    //其他
private:
    Log();
    virtual ~Log();
    //其他
```
静态成员instance只会被生成一次保证了单例。不过需要注意的是C++11之前的在创建``instance``需要加锁，因为编译器不保证内部静态变量的线程安全。C++11之后不需要了。  


log包含如下变量：
```C++
char dir_name[128]; //路径名
char log_name[128]; //log文件名
int m_split_lines;  //日志最大行数
int m_log_buf_size; //日志缓冲区大小
long long m_count;  //日志行数记录
int m_today;        //因为按天分类,记录当前时间是那一天
FILE *m_fp;         //打开log的文件指针
char *m_buf;
block_queue<string> *m_log_queue;   //阻塞队列
locker m_mutex;                     //互斥锁
int m_close_log;                    //关闭日志
```
提供了如下成员函数：


### ``bool init(const char *file_name,int close_log,int log_buf_size=8192,int split_lines,int max_queue_size)``


分为异步写入和同步写入两种方式,如果参数max_queue_size>=1表示采取异步。  
异步模式：  
初始化阻塞队列，并且创建一个线程（调用flush_log_thread）读取阻塞队列写日志。  
生成日志文件名（添加时间信息）。  
创建/打开日志文件。  


### ``void write_log(int level, const char *format, ...)``


生成日志写入条目，包括类型（INFO、ERROR）、时间等信息。
判断当前日期和类的日期是否一致，不一致则刷新该文件写入缓冲区并关闭之前的文件描述符，打开新的文件。  
判断异步写入还是同步写入：
如果同步，直接写入文件。  
如果异步，将代写入信息push进阻塞队列。


### ``void flush(void)``  


线程安全的刷新文件缓冲区（加互斥锁）。
### ``static void *flush_log_thread(void *args)``


提供给写日志工作线程的调用函数。这个函数调用私有成员函数``async_write_log()``


### ``void *async_write_log()``


异步写入日志函数，从阻塞队列取出日志条目，并写入日志。


## 宏定义


``log.h``中提供了几个宏定义，实现了对日志写入的包装，让其他函数能够更加轻松的调用日志类：
```C++
#define LOG_DEBUG(format, ...) if (0 == m_close_log){Log::get_instance()->write_log(0, format,##__VA_ARGS__);Log::get_instance()->flush();}
#define LOG_INFO(format, ...) if (0 == m_close_log){Log::get_instance()->write_log(1, format, ##__VA_ARGS__);Log::get_instance()->flush();}
#define LOG_WARN(format, ...) if (0 == m_close_log){Log::get_instance()->write_log(2, format, ##__VA_ARGS__);Log::get_instance()->flush();}
#define LOG_ERROR(format, ...)if (0 == m_close_log){Log::get_instance()->write_log(3, format, ##__VA_ARGS__);Log::get_instance()->flush();}
```


# 定时器


## client_data类


```C++
struct client_data
{
    sockaddr_in address;
    int sockfd;
    util_timer *timer;
};
```
客户数据类，数据成员有套接字描述符、地址、定时器。  


## util_timer类


```C++
class util_timer
{
public:
    util_timer() : prev(NULL), next(NULL) {}

public:
    time_t expire;  //定时器终止时间

    void (*cb_func)(client_data *);
    client_data *user_data;
    util_timer *prev;
    util_timer *next;
};
```
定时器类，数据成员包括前继结点、后继结点、终止时间、客户数据类。此类服务于定时器链表。


### ``cb_func(client_data *)``


功能：关闭已连接套接字，并设置epoll内核事件注册表（删除套接字）。  
此函数在定时器超时时候调用。


## sort_timer_lst类


升序定时器链表。  
按照终止时间升序排列的定时器双向链表。  
包括数据成员头指针、尾指针。  
提供公有成员函数：  
void add_timer(util_timer*)  
void adjust_timer(util_timer*)  
void del_timer(util_timer*)  
void tick()  


### ``add_timer(util_timer*)``

添加定时器到链表中

### ``adjust_timer(util_timer*)``

调整定时器在链表中的位置。由于事件的发生定时器终止时间可能会发生变化，也需要相应的调整其在链表中的位置。同时插入新的定时器，也需要调整位置

### ``del_timer(utile_timer*)``

删除定时器。

### ``tick()``

可以理解为滴答，即查看当前时间，将定时器链表中超时间的定时器对象删除，并调用``cb_func()``删除相应的注册事件和关闭套接字描述符。

## Utils类

这是一个工具类，直接面向服务器类``webserver``，负责管理连接的定时器链表、通信管道。
包含静态数据成员有：``int *u_pipefd``：这是管道，用于通信；``int u_epollfd``：这是epoll文件描述符。  包含数据成员：``sort_timer_lst m_timer_lst``：定时器链表；``int m_TIMESLOT``：时间间隔。
提供的成员函数：  
```C++
void init(int timeslot);
int setnonblocking(int fd);  
void addfd(int epollfd, int fd, bool one_shot, int TRIGMode);
static void sig_handler(int sig);
void addsig(int sig, void(handler)(int), bool restart = true);
void timer_handler();
void show_error(int connfd, const char *info);
```


### ``void init(int timeslot)``


设置定时器时间间隔

### ``int setnonblocking(int fd)``  

将文件描述符设置为非阻塞，即立即返回，不等待I/O。

### ``void addfd(int epollfd, int fd, bool one_shot, int TRIGMode)``  

向epoll内核注册表注册套接字描述符号，并设置触发模式LT/ET。one_shot用于设置同一套接字能否被多个线程处理。

### ``void sig_handler(int sig)``  

静态成员函数。信号处理函数：设计成将传递信号到管道。

### ``void addsig(int sig, void(handler)(int), bool restart = true)``  

为sig信号设置信号处理函数handler。

### ``void timer_handler()``  

定时任务处理，包括``tick()``、和发出SIGALRM信号。

### ``void show_error(int connfd, const char *info)``  

向套接字输出错误，一般只用于输出超时信息。

# 数据库连接池

``mysql/sql_connection_pool.h``、``mysql/sql_connection_pool.cc``申明了数据库连接池类``connection_pool``、数据库连接类``connectionRAII``。  
``connection_pool``提供数据库连接服务，先初始化好数据库连接，``connectionRAII``可以理解为前者的一个接口，只获得一个数据库连接，当http服务需要进行数据库连接时，只需要创建一个``connectionRAII``对象并从连接池获取连接即可。  

## connection_pool

单例模式实现，包含下列数据成员：  
```C++
private:
	int m_MaxConn;  //最大连接数
	int m_CurConn;  //当前已使用的连接数
	int m_FreeConn; //当前空闲的连接数
	locker lock;    //互斥锁
	list<MYSQL *> connList; //连接池
	sem reserve;
public:
	string m_url;           //主机地址
	string m_Port;	    	//数据库端口号
	string m_User;  		//登陆数据库用户名
	string m_PassWord;	    //登陆数据库密码
	string m_DatabaseName;  //使用数据库名
	int m_close_log;	    //日志开关
```
提供的成员函数包括：
```C++
	MYSQL *GetConnection();				 //返回一个数据库连接
	bool ReleaseConnection(MYSQL *conn); //释放连接
	int GetFreeConn();					 //获取连接
	void DestroyPool();					 //销毁所有连接

	static connection_pool *GetInstance();	//单例模式，获取唯一实例
	void init(string url, string User, string PassWord, string DataBaseName, int Port, int MaxConn, int close_log); //初始化连接池
```


## connectionRAII


数据成员包括：
```C++
private:
	MYSQL *conRAII;//数据连接
	connection_pool *poolRAII;//数据库连接池
```
成员函数有：
```C++
connectionRAII(MYSQL **con, connection_pool *connPool); 
//构造时直接从数据库连接池中获取数据库连接
~connectionRAII();
```


# HTTP服务


``http_connect.h``、``http_connect.cc``中实现了http服务类-http_connect(为连接提供http服务，一个连接对应一个http_connect对象），还是实现了``setnoblocking()``、``addfd()``、``removefd()``、``modfd()``等功能函数。  
上述功能函数用于设置epoll描述符的相关属性、向epoll内核事件表中添加/删除事件的功能。


## http_connect


本类提供以下成员函数

### ``void http_connect::initmysql_result(connection_pool* connPool)``

初始化对象的mysql连接，并将用户数据库数据读入到外部变量查找表users中。  

### ``void http_connect::close_conn()``

关闭一个连接

### ``void http_connect::init(int sockfd, const sockaddr_in &addr, char *root, int TRIGMode,int close_log, string user, string passwd, string sqlname)``

初始化连接：  
1. 将文件描述符添加到内核事件表中（addfd()）；  
2. 设置http响应方式；  
3. 设置日志是否关闭；  
4. 设置数据库信息；  
5. 调用init()初始化状态信息以及缓冲区。  

### ``void http_connect::init()``

初始化http_connect对象的各种信息以及缓冲区。  

### ``http_connect::LINE_STATUS http_connect::parse_line()``

状态机，用于分析出一行内容，返回值为行的读取状态，有LINE_OK,LINE_BAD,LINE_OPEN

### ``bool http_connect::read_once()``

读取客户数据，直到数据读取完毕或者无数据可读。  
1. 读缓冲区用完，返回false；  
2. 选择读取方式LT或者ET；  
3. 选择LT模式则只调用一次recv读取数据，并返回；  
4. 选择ET模式则循环调用recv直到读取完毕或者发生错误；  

### ``http_connect::HTTP_CODE http_connect::parse_request_line()``

读取http请求报文，并解析返回状态码。  

1. 检测是否是有效http请求，否则返回BAD_REQUEST；  
2. 读取url和http版本；  
3. 确认是否为主界面url（登陆/注册）；  
4. 修改请求行状态m_check_state为CHECK_STATE_HEADER并返回NO_REQUEST;

### ``http_connect::HTTP_CODE http_connect::parse_headers(char *text)``

解析一个http请求的首部行信息。  
1. 为空则返回GET_REQUESt；    
2. 匹配Connection（长连接或者短连接）；    
3. 匹配Content-length（实体部分长度）；  
4. 匹配Host（客户主机）；  
5. 如果前面无法匹配，则向日志写入信息；  
6. 返回NO_REQUEST。  

### ``http_connect::HTTP_CODE http_connect::parse_content(char *text)``

判断http请求是否被完整的读入缓冲区。

### ``http_connect::HTTP_CODE http_connect::process_read()``

1. 定位到读缓冲区内开始位置；  
2. 修改开始位置到m_checked_idx；  
3. 判断http请求的读取状态；  
4. 如果是正在解析请求行，则调用parse_request_line()；  
5. 如果是正在解析首部行，则调用parse_headers(text)，若返回GET_REQUEST则调用do_request()执行响应；  
6. 如果是正在解析实体，则调用parse_content()，若返回GET_REQUEST则调用do_request()执行响应，者之LINE_OPEN；  
7. 如果都不是，返回内部错误。  

### ``void http_connect::unmap()``

释放读入到内存的被请求资源。

### ``bool http_connect::add_reponse(const char *format, ...)``

向要返回的报文中添加响应，即写入缓冲区中。
还有相关函数``add_status_line()``、``add_headers()``、``add_content()``，分别表示写入状态行，写入首部行，写入实体。

### ``bool http_connect::process_write(HTTP_CODE ret)``

写事件处理函数，通过调用本函数来实现对返回HTTP报文的写。

### ``void http_connect::process()``

读取或写入数据，并向内核事件表注册IO就绪事件。


# 线程池


``threadpool.h``实现了threadpool类，该类能够提供多线程的http服务。

## threadpool类

提供了如下成员函数：

### ``threadpool::threadpool(...)``

构造函数，初始化对象参数并且创建线程(将线程设置成分离的)。

### ``threadpool::~threadpool()``

释放线程数组资源。

### ``bool threadpool::append_reactor(http_connect *request, int state)``

将一个http_connect对象添加到请求队列中（Reactor模式）。

### ``bool threadpool::append_proactor(http_connect *request)``

将一个http_connect对象添加到请求队列中（Proactor模式）。


### ``void *threadpool::worker(void* arg)``

线程函数，调用run()函数。

### ``void threadpool::run()``

1. 从请求队列读取http_connect对象；  
2. 判断事件处理模式；  
3. 如果是Reactor模式则进行I/O操作并向内核注册写事件就绪；  
4. 如果是Proactor模式则调用process()处理。


# 服务器


``webserver.h``、``webserver.cc``定义并且实现了服务器类webserver，该类是本项目最核心的类，所有类围绕webserver提供服务。

# webserver类


### ``void init(int,string,string,string,int,int,int,int,int,int,int)``

初始化相关参数，包括服务器端口、数据库用户名、数据库密码等等。

### ``void thread_pool()``  

初始化线程池

### ``void sql_pool()``

初始化数据库连接池

### ``void log_write()``  

初始化日志类，并且打开一个日志文件

### ``void trig_mode``  

设置监听套接字和已连接套接字的触发模式（LT/ET）

### ``void eventListen()``

事件监听函数：  
1）打开一个监听套接字  
2）设置这个监听套接字的相关参数  
3）设置Uilts类成员utils  
4）创建epoll内核事件表  
5）向utils添加套接字事件（包括设置定时器和将套接字和epoll关联起来）  
6）创建通信管道，并将读管道和epoll关联起来  
7）将管道变量、epoll套接字描述符赋值给Utils  
8）设置信号SIGPIPE、SIGALRM、SIGTERM的处理
9）发出SIGALRM信号

### ``void eventLoop()``

事件循环函数。  
1）调用``epoll_wait()``等待IO事件发生。  
2）处理事件：  
如果是监听套接字，则处理监听数据。  
如果对应文件描述符断开连接/发生错误，则阐述对应的管理类（定时器），调用deal_timer()。  
如果是信号管道的读端口且有可读事件发生，则调用信号处理函数，并返回服务器停止、超时信息。  
如果是读事件，处理客户连接上的读事件（dealwithdata）。  
如果是写事件，处理客户连接上的写事件（dealwithwrite）。  
3）如果超时，则调用utils的定时任务，并输出日志信息

### ``void timer(int connfd, struct sockaddr_in，client_address)``  

1）初始化套接字对应的http连接类  
2）初始化定时器，并加入定时器链表

### ``void adjust_timer(util_timer *timer)``

修改定时器终止时间，调整定时器在链表中的位置。

### ``void deal_timer(util_timer *timer, int sockfd)``

删除定时器，关闭套接字。

### ``bool dealclinetdata()``

1）接收客户端连接，其中有LT/ET模式可选。  
2）在http连接管理类、定时器管理类中初始化这个连接。

### ``bool dealwithsignal(bool &timeout, bool &stop_server)``

1）从管道读取信号到缓冲区。  
2）处理SIGARM信号（返回timeout信息）  
3）处理终止信号（返回stop_server信息）

### ``void dealwithread(int sockfd)``

提供了Reactor和Proacotr两种事件处理模式。  
Reactor：   
调整定时器。  
将事件放入请求队列。    
Proactor：  
读取数据，若读取失败调用deal_timer()，读取成功则将事件添加进入请求队列，调整定时器（adjust_timer）。

### ``void dealwithwrite(int sockfd)``

提供了Reactor和Proacotr两种事件处理模式。  
Reactor：   
调整定时器。  
将事件放入请求队列。    
Proactor：  
读取数据，若读取失败调用deal_timer()，读取成功则将事件添加进入请求队列，调整定时器（adjust_timer）。



# 压力测试



服务器项目仅仅只是完成项目还是不够的，我们还需要进行压力测试来检验这个服务器的性能如何，只要经过了压力测试，才说说明这个服务器是稳定的。


## Webbench

在本项目中我使用Webbench来进行压力测试。Webbench是知名的网站压力测试工具，它是由[Lionbridge公司](http://www.51testing.com)开发。Webbench能测试处在相同硬件上，不同服务的性能以及不同硬件上同一个服务的运行状况。webbench的标准测试可以向我们展示服务器的两项内容：每秒钟相应请求数和每秒钟传输数据量。webbench不但能具有便准静态页面的测试能力，还能对动态页面（ASP,PHP,JAVA,CGI）进 行测试的能力。还有就是他支持对含有SSL的安全网站例如电子商务网站进行静态或动态的性能测试。Webbench最多可以模拟3万个并发连接去测试网站的负载能力。