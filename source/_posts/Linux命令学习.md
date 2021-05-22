title: Linux命令学习
author: Jiahao Wu
tags:
  - Linux
categories:
  - Linux C/C++
date: 2021-02-15 22:19:00
---
# 网络相关

## tcpdump

``tcpdump``是一个抓包命令。有如下参数：  

-**i**:指定需要抓包的网卡。如果未指定的话，tcpdump会默认选择搜索到的系统中状态为up的最小数字的网卡，一般情况下是eth0。-i lo抓取回环的数据包。  
-**nnn**：禁用tcpdump展示时把IP、端口等转换为域名、端口知名服务名称。这样看起来会更清晰  
-**s**：指定抓包的包大小。使用-s 0指定数据包大小为262144。可以使抓到的数据包不被截断，完整反映数据包的内容。默认68字节。  
-**c**：指定抓包的数量。  
-**w**：指定抓包结果保存的文件，以便后续用wireshark等工具进行分析。  
-**P**：in/out/inout，指定要抓取的包是流入还是流出的包。默认inout  
-**F**：使用文件作为过滤条件表达式的输入, 此时命令行上的输入将被忽略.  
-**p**： 一般情况下, 把网络接口设置为非'混杂'模式。  


# 进程、线程相关


## ps

``ps``可以用来查看当前有哪些进程，涉及很多参数，不一一列举，要用再查。  
加上``-T``选项可以开启线程查看，比如下面命令可以查看进程``<pid>``创建的所有线程：
```Linux
  Linux> ps -T -p <pid>
```
  
## top

``top``可以实时查看各个线程的情况，以下命令可以查看Linux所有线程：
```Linux
Linux> top -H
```

而以下命令可以只查看进程``<pid>``的所有线程：
```Linux
Linux> top -H -p <pid>
```

## strace

``strace``是用来跟踪进程的系统调用的，可以用如下命令跟踪进程``<pid>``的所有系统调用并将结果输出到output.txt文件中：
```Linux
Linux> sudo strace -o output.txt -p <pid>
```
``strace``还有很多其他参数，比如``-e expr``，expr是一个表达式，比方说``-e trace=open``就是跟踪系统调用open。参数很多，这里不一一列举了。  

## pstack

``pstack``使用来跟踪进程的栈的命令。如下命令可以查看进程``<pid>``的栈。  
```Linux
Linux> pstack <pid>
```

## memstat

``memstat``可以用来查询共享库的信息，命令如下：
```Linux
Linux> memstat -w | sort -rn
// -w标识全部打印（没有会截断长度超过80的字符串），然后按照字节数来排序
```

## free

``free``可以显示内存情况，可用参数有：  
- b，以字节为单位显示；  
- k，以kb为单位显示；  
- m，以mb为单位显示；  
....
不一一列举，需要再查。

# 网络状态和防火墙

## netstat

``netstat``命令用于显示Linux系统网络链接状态。netstat命令用于显示与IP、TCP、UDP和ICMP协议相关的统计数据，一般用于检验本机各端口的网络连接情况。如果我们要显示当前的TCP连接，可以这样：
```Linux
Linux> netstat --tcp
```

## ifconfig

这个命令就是用来显示或设置网络设备：
```Linux
Linux> ifconfig
Linux> ifconfig eth0 down
Linux> ifconfig eth0 up
```











