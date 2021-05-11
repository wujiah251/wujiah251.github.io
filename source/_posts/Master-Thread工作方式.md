---
title: Master Thread工作方式
date: 2021-05-09 22:11:20
tags:
---

# Master Thread工作方式

Master Thread具有最高级别的线程优先级。其内部又多个循环组成：

- 主循环（loop）

- 后台循环（backgroup loop）
- 刷新循环（flush loop）
- 暂停循环（suspend loop）

Loop被称为主循环是因为大部分操作是在这个循环中完成的。包括每秒钟的操作和每十秒钟的操作：

每秒一次的操作：

- 日志刷新到磁盘（即使这个事务还没有提交）；
- 合并插入缓冲（可能）；
- 至多刷新100个InnoDB的缓冲池中脏页到磁盘；
- 如果当前没有用户活动，则切换到backgroud loop、

每十秒的操作：

- 刷新100个脏页到磁盘；
- 合并至多5个插入缓冲；
- 将日志缓冲刷新到磁盘；
- 删除无用的Undo页；
- 刷新100个或者10个脏页到磁盘；

若当前没有用户活动或者数据库关闭，就会切换到backgroud loop：

- 删除无用的undo 页面；
- 合并20个插入缓冲；
- 跳回到主循环；
- 不断刷新100个页直到符合条件（可能，跳转到flush loop中完成）。

若flush loop 中也没有什么事情可以做，InnoDB会切换到suspend loop，将Master Thread挂起，等待时间发生。

## InnoDB 1.2.x版本的Master Thread

在InnoDB 1.2.x版本中对Master Thread进行了优化，将对于刷新脏页的操作从Master线程分离到了一个单独的Page Cleaner Thread中，减轻了Master Thread的工作。

