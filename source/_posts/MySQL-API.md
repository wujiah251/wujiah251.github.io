title: MySQL API
author: Jiahao Wu
tags: []
categories:
  - MySQL学习
date: 2021-01-22 23:14:00
---
常见的MySQL API函数如下所示：
```C++
MYSQL *mysql_real_connect (MYSQL *mysql,
const char *host,
const char *user, 
const char *passwd, 
const char *db, 
unsigned int port,
const char *unix_socket,
unsigned long client_flag);
// 返回一个MYSQL指针，host未null或者localhost时链接本地的计算机
// user和passwd是账号和密码
// db是数据库名称，port是端口，MySQL服务器默认端口号是3306
// unix_socket为null标识不适用socket或管道机制，最后一个参数经常设置为0

int mysql_select_db(MYSQL *mysql,
const char *db);
// 选择一个数据库，db是数据库名，类似 USE

int mysql_query(MYSQL *mysql, const char *q);
// 建立一个查询，q是查询语句，C风格字符串，内容语法和MySQL一致

MYSQL_RES *mysql_store_result(MYSQL *mysql);
// 将全部数据保存在本机
// 返回一个结果指针

unsigned int mysql_num_fields(MYSQL_RES *res);
// 返回结果集的行数

MYSQL_FIELD *mysql_fetch_fields(MYSQL_RES *res);
// 对于结果集，返回所有MYSQL_FIELD结构的数组。每个结构提供了结果集中1列的字段定义。

MYSQL_ROW mysql_fetch_row(MYSQL_RES *result);
// 检索结果集的下一行。在mysql_store_result()之后使用时，如果没有要检索的行，mysql_fetch_row()返回NULL。

void mysql_close(MYSQL *sock);
// 关闭一个数据库连接

```
