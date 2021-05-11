---
title: InnoDB关键特性
date: 2021-05-09 22:35:48
tags:
---

# InnoDB关键特性

- 插入缓冲（Insert Buffer）
- 两次写（Double Write）
- 自适应哈希索引（Adaptive Hash Index）
- 异步IO（Async IO）
- 刷新邻接页（Flush Neighbor Page）

## 插入缓冲

### Insert Buffer

Insert Buffer并不是缓冲池的一部分，而是物理页的一个组成部分（不过缓冲池中有Insert Buffer的信息）。

对于非聚集索引的插入或者更新操作，不是每一次直接插入到索引页中，而是先判断插入的非聚集索引页是否在缓冲池中。若在，则直接插入；若不在，则先放入到一个Insert Buffer对象中，好似欺骗。数据库这个非聚集的索引已经插到了叶子节点，而实际并没有，只是存放在另一个位置。然后以一定频率合并到一个操作中。

使用Insert Buffer的使用需要同时满足以下两种情况：

- 索引是辅助索引；
- 索引不是唯一；

问题：如果服务器宕机了，可能会有大量的Insert Buffer并没有合并到实际的非聚集索引中去。

### Change Buffer

Insert Buffer是一颗B+树，因此其也由叶节点和非叶节点组成。非叶节点存放的是查询search key，构造如下：

|space|marker|offset|。

space占用4字节，标识待插入记录所在表的表空间id。

marker占一个字节，用来兼容老版本的Insert Buffer。

offset标识页所在的偏移量，占用4字节。

当一个辅助索引要插入到页时，如果这个页不再缓冲池中，那么InnoDB存储引擎首先根据上述规则构造一个search key，接下来查询Insert Buffer这棵B+树，然后将这条记录插入到Insert Buffer B+树的叶子节点（所有表共享一个insert buffer）。

### Merge Insert Buffer

若需要实现插入记录的辅助索引在缓冲池中，那么需要将辅助索引记录首先插入到这棵B+树中。但是Insert Buffer中记录何时合并到真正的辅助索引中呢？这是本小节需要关注的重点。

Merge可能会发生在以下三种情况：

- 辅助索引页被读取到缓冲池时；
- Insert Buffer Bitmap页追踪到该辅助索引页已无可用空间时；
- Master Thread。











