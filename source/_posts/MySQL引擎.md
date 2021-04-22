title: MySQL引擎
author: Jiahao Wu
tags: []
categories:
  - MySQL学习
date: 2021-02-26 17:35:00
---
MySQL存储引擎有InnoDB、MyISAM、MEMORY。

# InnoDB

InnoDB是事务型数据库的首选引擎，支持事务安全（ACID），支持行锁定和外键，InnoDB是MySQL默认的引擎。  
主要特性：
1. InnoDB提供了具有提交、回滚和崩溃恢复的事务安全存储引擎。  
2. InnoDB存储引擎在主内存中缓存数据和索引而维持它的缓冲池。它表和索引放在一个逻辑表空间中，而MyISAM表中每个表被存放在分离的文件中。  
3. InnoDB支持外键完整性约束。  

# MyISAM

MyISAM基于ISAM，是在Web、数据仓储和其他应用环境下最常使用的存储引擎之一。MyISAM拥有较高的插入、查询，但不支持事务。  
主要特性：  
1. 查询、插入效率高；  
2. 可以把数据文件和索引文件放在不同的目录中；  
3. 对事务的支持不好。  

# MEMORY

MEMORY存储引擎将表中的数据存储到内存中，为查询和引用其他表数据提供快速访问。  