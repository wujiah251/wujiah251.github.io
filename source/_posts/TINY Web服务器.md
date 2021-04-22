title: TinyWeb 服务器
author: Jiahao Wu
tags:
categories: 深入理解计算机系统
---
# TinyWeb 服务器

我针对原来提供的代码，仔细阅读，附上注释版（每个函数的功能书中有阐释，就不细讲，可以结合着看）：
## main函数及其他函数定义
```C++
// 这是一个微型Web服务器，能够提供5种常见类型的静态内容：HTML文集爱你、无格式的>文本文件、以及编码为GIF、PNG和JPG格式的图片

// 函数库csapp.h
#include "csapp.h"
void doit(int fd);
// 这是函数负责处理HTTP事务
void read_requesthdrs(rio_t *rp);
// 读取报头
int parse_uri(char* uri,char *filename,char *cgiargs);
// 解析uri
void serve_static(int fd, char *filename,int filesize);
// 静态内容服务
void get_filetype(char* filename,char* filetype);
// 获得文件类型，html、plain、jpg、png、gif
void serve_dynamic(int fd, char *filename, char *cgiargs);
// 动态内容服务
void clienterror(int fd, char *cause, char *errnum, char *shortmsg, char* longmsg);
// 向客户端报错


int main(int argc,char **argv){
    int listenfd, connfd, port, clientlen;
    struct sockaddr_in clientaddr;

    if(argc!=2){
        // argc==2表示输入为文件名 和 端口号
        // 调用方式为 >./mytiny 1024
        fprintf(stderr, "usage: %s<port>\n", argv[0]);
        exit(1);
    }
    port = atoi(argv[1]);//字符串转数字
    listenfd = Open_listenfd(port);
    while(1){
        clientlen = sizeof(clientaddr); //地址结构体的字节数
        connfd = Accept(listenfd,(SA*)&clientaddr, &clientlen); //产生一个已连接
套接字
        doit(connfd);//执行http事务处理函数
        Close(connfd);//关闭已连接套接字connfd-关闭连接
    }
}

```









## doit函数
```C++
void doit(int fd){
    // 输入fd是一个已连接套接字描述符
    int is_static;
    char filename[MAXLINE],cgiargs[MAXLINE];
    // uri解析出来的路径和cgi参数
    struct stat sbuf; // stat结构体是用于存储文件信息的类型
    char buf[MAXLINE], method[MAXLINE], uri[MAXLINE], version[MAXLINE];
    // buf-缓冲区，method-方法(比如GET)，uri-URI，version-http的版本
    rio_t rio;
    Rio_readinitb(&rio, fd); // 初始化创建一个空的读缓冲区并且和套接字描述符fd关
联起来
    Rio_readlineb(&rio, buf, MAXLINE); //读取一行
    printf("Request headser\n:");
    printf("%s",buf);
    sscanf(buf,"%s %s %s",method, uri,version);
    // 从缓冲区去读命令到变量

    if(strcasecmp(method,"GET")){
        // 不想等返回非0值
        // 调用clienterror向客户端返回错误，不支持非GET方法
        clienterror(fd, method, "501", "Not Implemented", "Tiny does not implement this method");
        return;
    }
    read_requesthdrs(&rio); //读取请求报文

    is_static = parse_uri(uri, filename, cgiargs);
    // 如果请求的是静态内容返回1，否则返回0
    // filename用来存储解析出来的路径
    //  cgiargs是一个可选的CGI参数字符串，如果是静态内容则清空
    if(stat(filename,&sbuf)<0){
        // 文件信息获取成功返回0，失败返回-1
        clienterror(fd, filename, "404", "Not found","Tiny couldn't find this file");
        return;
    }

    if(is_static){
        if(!(S_ISREG(sbuf.st_mode)) || !(S_IRUSR & sbuf.st_mode)){
            // st_mode：保护模式
            // S_ISREG(st_mode)用于判断是不是一个常规文件
            // S_IPUSR用户读权限
            clienterror(fd,filename, "403", "Forbidden", "Tiny couldn't read the file");
            // 403：服务器无权访问所请求程序
            return;
        }
        serve_static(fd, filename, sbuf.st_size);//提供静态响应函数
        // st_size：文件大小
    }
    else{
        if(!(S_ISREG(sbuf.st_mode)) || !(S_IXUSR & sbuf.st_mode)){
            clienterror(fd, filename, "403", "Forbidden", "Tiny couldn't read the file");
            // 无权访问
            return;
        }
        serve_dynamic(fd,filename,cgiargs); //提供动态响应的函数
    }
    return;
}

```



## read_requesthdrs函数
```C++
void read_requesthdrs(rio_t *rp)
{
   char buf[MAXLINE];
   Rio_readlineb(rp,buf,MAXLINE);
   while(strcmp(buf, "\r\n")){
        //终止请求报头是\r\n组成的，相等返回0，否则返回非0
        Rio_readlineb(rp,buf,MAXLINE);
        printf("%s",buf);
   }
   return;
}
```


## parse_uri函数
```C++
int parse_uri(char *uri,char *filename, char *cgiargs)
{
    char *ptr;
    if(!strstr(uri,"cgi-bin")){
        // 没有匹配到则返回null
        strcpy(cgiargs, ""); //清空cgiargs
        strcpy(filename,"."); // 变成相对路径
        strcat(filename,uri); // 比如./intex.html
        if(uri[strlen(uri)-1]=='/')//如果uri以/为结尾，则在路径末尾再添加home.html
            strcat(filename,"home.html");
        return 1;
    }
    else{
        ptr = index(uri,'?');
        // 索引到?，?之后是参数
        if(ptr){
            strcpy(cgiargs,ptr+1); //将参数读入cgiargs
            *ptr = '\0'; //避免把参数读入到filename
        }
        else{
            strcpy(cgiargs,"");
        }
        strcpy(filename,".");
        strcat(filename,uri);
        return 0;
    }
}
```


## serve_static函数
```C++
void serve_static(int fd, char *filename, int filesize)
{
    // 提供5种常见类型的静态内容：HTML文件、无格式文本文件、编码为GIF、PNG、JPG>格式的图片文件

    int srcfd;
    char *srcp, filetype[MAXLINE], buf[MAXBUF];
    // 发送响应到客户端

    get_filetype(filename, filetype); //获得文件类型
    // 将HTTP报文写到buf缓存区
    sprintf(buf, "HTTP/1.0 200 OK\r\n"); //初始状态行
    // 首部行
    sprintf(buf, "%sServer: Tiny Web Server\r\n",buf);
    sprintf(buf, "%sContent-length: %d\r\n", buf, filesize); //长度
    sprintf(buf, "%sContent-type: %s\r\n\r\n", buf, filetype); //类型
    Rio_writen(fd, buf, strlen(buf));
    printf("Response headers:\n");
    printf("%s",buf);

    srcfd = Open(filename, O_RDONLY, 0);//只读打开文件
    srcp = Mmap(0, filesize, PROT_READ, MAP_PRIVATE, srcfd, 0);
    // Mmap函数在csapp.h、csapp.c中定义、实现
    // 这只是mmap的一个包装函数
    // 功能是将分见映射到内存，用内存读写取代I/O读写
    // 第1个变量void* start，指向内存起始的位置，通常设为NULL(其实就是(void*)0)>，代表让系统自动选定地址
    // 第2个变量size_t length，将文件多大部分映射到内存中
    // 第3个变量int prot，表示映射区域的保护方式，这里使用可读
    // 第4个变量int flags，表示影响映射区域的各种特性，MAX_PRIVATE是私人的写时复
制
    // 第5个变量int fd，文件描述符
    // 第6个变量off_t offsize文件映射的偏移量，0表示从文件的最前方开始
    // 返回值为映射区的内存起始位置
    Close(srcfd); //关闭打开文件
    Rio_writen(fd, srcp, filesize);
    Munmap(srcp, filesize);//解除内存映射
}

```




## get_filetype函数
```C++
void get_filetype(char *filename, char *filetype)
{
    // 获取文件类型
    if(strstr(filename, ".html"))strcpy(filetype, "text/html");
    else if(strstr(filename, ".gif"))strcpy(filetype, "image/gif");
    else if(strstr(filename, ".png"))strcpy(filetype, "image/png");
    else if(strstr(filename, ".jpg"))strcpy(filetype, "image/jpeg");
    else strcpy(filetype, "text/plain");
}
```

## serve_dynamic函数
```C++
void serve_dynamic(int fd, char *filename, char *cgiargs)
{
    char buf[MAXLINE], *emptylist[]={NULL};
    // 响应报文
    sprintf(buf, "HTTP/1.0 200 OK\r\n");
    Rio_writen(fd, buf, strlen(buf));
    sprintf(buf, "Server: Tiny Wev Server\r\n");
    Rio_writen(fd, buf, strlen(buf));

    if(Fork()==0){
        setenv("QUERY_STRING", cgiargs, 1);
        //int setenv(const char *name,const char * value,int overwrite);
        //setenv()用来改变或增加环境变量的内容
        //参数name为环境变量名称字符串；value为变量内容，参数overwrite用来决定是否要改变已存在的环境变量。如果 overwrite不为0，而该环境变量原已有内容，则原内容>会被改为参数value所指的变量内容。如果overwrite为0，且该环境变量已有内容，则参数value会被忽略。
        //执行成功返回0，否则-1
        Dup2(fd, STDOUT_FILENO);
        // Dup2是csapp对dup2的封装，原型dup2(int oldfd,int newfd)
        // 复制文件描述符oldfd，并且指定新的文件描述符数值为newfd
        // STDOUT_FILENO是linux提供的用于标准出入的文件描述符
        Execve(filename, emptylist, environ);
        // 对execve的包装，原型：execve(const char *filename, cahr *const argv[],char *const envp[]);

    }
    Wait(NULL);
}
```

## clienterror函数
```C++
void clienterror(int fd, char *cause, char *errnum, char *shortmsg, char *longmsg)
{
    char buf[MAXLINE], body[MAXBUF];
    // 直接复制了，懒得写
    /* Build the HTTP response body */
    sprintf(body, "<html><title>Tiny Error</title>");
    sprintf(body, "%s<body bgcolor=""ffffff"">\r\n", body);
    sprintf(body, "%s%s: %s\r\n", body, errnum, shortmsg);
    sprintf(body, "%s<p>%s: %s\r\n", body, longmsg, cause);
    sprintf(body, "%s<hr><em>The Tiny Web server</em>\r\n", body);

    /* Print the HTTP response */
    sprintf(buf, "HTTP/1.0 %s %s\r\n", errnum, shortmsg);
    Rio_writen(fd, buf, strlen(buf));
    sprintf(buf, "Content-type: text/html\r\n");
    Rio_writen(fd, buf, strlen(buf));
    sprintf(buf, "Content-length: %d\r\n\r\n", (int)strlen(body));
    Rio_writen(fd, buf, strlen(buf));
    Rio_writen(fd, body, strlen(body));
}
```







## 使用方式
首先创建一个目录：mytiny，然后在mytiny中创建一个文件mytiny.c，然后将csapp.h(在include中)、csappp.c(在src中)复制到此目录中。
首先``gcc -c csapp.c``获得csapp.o文件，然后``ar -rc csapp.a csapp.o``生成静态库文件。
生成可执行文件``gcc -o mytiny mytiny.c csapp.a``。
在mytiny中创建目录cgi-bin，将``netp/tiny/cgi-bin/adder.c``复制到此目录中，然后csapp.a、csapp.h也复制一份到其中。
然后运行命令``gcc -o adder adder.c csapp.a``生成可执行文件。
为了测试对图片的请求，我们添加文件test.jpg在mytiny中。
我们最终的目录结构如下：
```
mytiny/
    mytiny.c
    csapp.h
    csapp.c
    csapp.o
    csapp.a
    mytiny.o
    test.jpg
    cgi-bin/
        adder.c
        csapp.h
        csapp.a
        adder.o
```
### telnet测试
首先：``linux> ./mytiny 12000``
首先使用TELNET测试，打开另一个shell，``linux> telnet localhost 12000``
然后输入HTTP请求行：``GET /cgi-bin/adder?100&20 HTTP/1.0``。输完后需要先后回车两次。
然后我们可以看到HTTP响应。

### 浏览器测试
照样先运行服务器：``linux> ./mytiny 12000``。
在浏览器中输入``http://localhost:12000/test.jpg``，然后我们可以看到图片。
