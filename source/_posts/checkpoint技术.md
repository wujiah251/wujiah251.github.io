---
title: checkpoint技术
date: 2021-05-09 21:37:30
tag:
---

# Checkpoint技术

缓冲池的设计目的是为了协调CPU速度于磁盘速度的鸿沟。因此页的操作首先都是在缓冲池中完成的。

当前事务数据库系统普遍采用Write Ahead Log策略，即当事务提交时，先写日志，再修改页。当由于发生宕机而导致数据丢失时，通过重做日志来完成数据恢复。

Checkpoint（检查点）技术的目的是解决以下几个问题：

- 缩短数据库的恢复时间；

- 缓冲池不够用时，将脏页刷新道磁盘；
- 重做日志不可用时，刷新脏页；

当数据库发生宕机时，数据库不需要重做所有日志，因为Checkpoint之前的页已经刷新回磁盘。故数据库只需对Checkpoint后的重做日志进行恢复。

缓冲池不够用时，根据LRU算法回溢出最近最少使用的页，若此页为脏页则需要强制执行Checkpoint。

当数据库宕机后恢复操作会有一部分不需要的重做日志，这部分就可以被覆盖使用了，若此时重做日志还需要使用，那么必须强制产生Checkpoint，将缓冲区中的页至少刷新道当前重做日志的位置（重做日志中在最后一次checkpoint之后的内容是没有刷新到磁盘的，所以不可被覆盖）。

InnoDB是通过LSN来标记版本的，每个页有LSN，重做日志也有LSN，Checkpoint也有LSN。

两种Checkpoint，分别是：

- Sharp Checkpoint
- Fuzzy Checkpoint

Sharp Checkpoint发生的数据库关闭时将所有的脏页都刷新回磁盘。但是我们不能在运行时使用sharp checkpoint，这样数据库的可用性就会下降。

在InnoDB存储引擎内部使用Fuzzy Checkpoint进行页的刷新，即只刷新一部分脏页，而不是刷新所有的脏页回磁盘。

## InnoDB中可能的几种Checkpoint

- Master Thread Checkpoint
- FLUSH_LRU_LIST Checkpoint
- Async/Sync FLush Checkpoint
- Dirty Page too much Checkpoint

Master Thread是在主线程中发生的Checkpoint，差不多每秒或每十秒从缓冲区的**脏页列表**中刷新一定比例的页回磁盘。这个过程是异步的，即此时InnoDB存储引擎可以进行其他的操作，用户查询线程不会阻塞。

FLUSH_LRU_LIST Checkpoint是因为InnoDB存储引擎需要保证LRU列表中需要有差不多100各空闲页可供使用。检查是否有可用LRU的时候，如果发现用完，会将LRU列表的尾端的页溢出，如果是脏页，那么需要进行Checkpoint。

Async/Sync Flush Checkpoint指的是重做日志不可用的情况，这时需要强制将一些页刷新回磁盘最新页的LSN记为checkpoint_lsn。

最后一种就是Dirty Page too much，即脏页太多了，导致InnoDB存储引擎强制进行Checkpoint。

