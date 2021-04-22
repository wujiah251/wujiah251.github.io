title: 使用MySQL
author: Jiahao Wu
categories: MySQL学习
date: 2021-01-26 23:15:17
tags:
---
# 使用MySQL

## 连接

```Linux
Linux> mysql -u xxx -p
//xxx是用户名
```
然后再输入密码，就可以连接上本地的MySQL服务器了。端口使用默认端口3306。  

## 选择数据库

**关键字（key word）**作为MySQL语言组织的一个保留字。不要用关键字命名一个表或列。常见的关键字有``use``、``select``、``where``等等。  
选择数据库：
```MySQL
mysql> use crashcource;
Database changed
```


## 了解数据库和表

以下命令可以用来展示当前MySQL服务器中有哪些数据库：
```MySQL
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mydb               |
| mysql              |
| performance_schema |
| study              |
| sys                |
| webserver_database |
+--------------------+

```
以下命令可以用来查询当前数据中有哪些表（前提是你已经连接上了数据库）：
```MySQL
mysql> show tables;
+-----------------+
| Tables_in_study |
+-----------------+
| customers       |
| orderitems      |
| orders          |
| productnotes    |
| products        |
| vendors         |
+-----------------+
```
这个数据库内的表是用[MySQL必知必会官方网站](https://forta.com/books/0672327120/)上提供的sql脚本``create.sql``、``populate.sql``生成的。生成方式如下（你需要先将文件下载下来）。
```
mysql> create database XXX;
// XXX是你创建的数据库，名字你可以任意定义
mysql> use XXX;
mysql> source create.sql;
// 此文件创建表
mysql> source populate.sql; 
// 创建完成了
```
如下方法可以显示表中所有可用的列，以及相关参数：
```MySQL
mysql> show columns from customers;
+--------------+-----------+------+-----+---------+----------------+
| Field        | Type      | Null | Key | Default | Extra          |
+--------------+-----------+------+-----+---------+----------------+
| cust_id      | int(11)   | NO   | PRI | NULL    | auto_increment |
| cust_name    | char(50)  | NO   |     | NULL    |                |
| cust_address | char(50)  | YES  |     | NULL    |                |
| cust_city    | char(50)  | YES  |     | NULL    |                |
| cust_state   | char(5)   | YES  |     | NULL    |                |
| cust_zip     | char(10)  | YES  |     | NULL    |                |
| cust_country | char(50)  | YES  |     | NULL    |                |
| cust_contact | char(50)  | YES  |     | NULL    |                |
| cust_email   | char(255) | YES  |     | NULL    |                |
+--------------+-----------+------+-----+---------+----------------+
```