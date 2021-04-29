title: AOF持久化
author: 乡村程序员
tags: []
categories:
  - Redis
date: 2021-04-29 23:59:00
---
# AOF持久化

与RDB持久化通过保存数据库中的键值对来记录数据库状态不同，AOF持久化是通过保存Redis服务器所执行的写命令来记录数据库状态的。

## AOF持久化的实现

当AOF持久化功能处于打开状态时，服务器执行完一个写命令之后，会以协议格式将被执行的写命令追加到服务器状态的aof_buf缓冲区末尾。

### AOF文件的写入和同步

Redis服务器就是一个时间循环，这个循环中的文件事件负责接收客户端的命令请求，以及向客户端发送命令回复，而时间事件则负责执行像serverCron函数这样需要定时运行的函数。

服务器在每次结束一个时间循环之前，它都会调用flushAppendOnlyFile函数，考虑是否需要将aof_buf缓冲区内容写入和保存在AOF文件里面。

```python
def eventLoop():
	while True:
		processFileEvents()
		processTimeEvents()
		flushAppendOnlyFile()
```

flushAppendOnlyFile()函数的行为由服务器配置的appendfsync选项的值来决定，各个不同值产生的行为如表所示：

## AOF文件的载入与数据还原

AOF文件中包含了重建数据库状态的所需所有写命令，所以服务器只需要读入并重新执行一遍AOF文件里面保存的写命令，就可以还原服务器关闭之前的数据库状态。详细步骤：

1. 创建一个不带网络连接的伪客户端（因为Redis命令只能在客户端上下文中执行）。
2. 从AOF文件中分析并读取出一条写命令。
3. 使用伪客户端执行被读出的命令。
4. 一直执行步骤2和步骤3，直到AOF文件中所有的写命令都被处理完毕为止。

## AOF重写

为了解决AOF体积膨胀的问题，Redis提供了AOF文件重写功能。通过该功能，Redis服务器可以创建一个新的AOF文件来替代线有的AOF文件，新旧两个AOF文件所保存的数据库状态相同，但新AOF文件不会包含任何浪费空间冗余命令。所以新AOF文件的体积通常会比旧AOF文件的体积要小很多。

### AOF文件重写的实现

虽然Redis服务器将生成新的AOF文件替换旧的AOF文件的功能命名为AOF重写。但实际上，AOF文件重写并不需要对现有的AOF文件进行任何读取、分析或者写入操作。这个功能实际上是通过读取服务器当前的数据库状态来实现。伪代码如下：

```python
def aof_write(new_aof_file_name):
  # 创建新的AOF文件
  f = create_file(new_aof_file_name)
  # 遍历数据库
  for db in redisServer.db:
    # 忽略空数据库
    if db.is_empty():continue
    # 写入select命令，指定数据库号码
    f.write_command("SELECT" + db.id)
    # 遍历数据库中所有的键
   	for key in db:
      if key.is_expired():continue
      # 根据键的类型对键进行重写
      if key.type == String:
        rewrite_string(key)
      elif key.type == List:
        rewrite_list(key)
      elif key.type == Hash:
        rewrite_hash(key)
      elif key.type == Set:
        rewrite_set(key)
      elif key.type == SortedSet:
        rewrite_sorted_set(key)
      # 如果键带有过期时间，那么过期时间也要被重写
      if key.have_expire_time():
        rewrite_expire_time(key)
 	# 写入完毕，关闭文件
  f.close()
```

因为aof_rewrite函数所生成的新的AOF文件只包含还原当前数据库状态所必须的命令，所以新的AOF文件不会浪费任何硬盘空间。

### AOF后台重写

上面介绍的AOF重写程序aof_write函数可以很好地完成创建一个新的AOF文件的任务，但是因为这个函数会进行大量的写入操作，所以调用这个函数的线程将长时间阻塞，因为Redis服务器使用单线程来处理命令请求，所以服务器直接调用aof_rewrite函数的话，那么在重写AOF文件期间，服务将无法处理客户端发来的命令请求。

很明显作为一种辅佐性的维护手段，Redis不希望AOF重写造成服务器无法处理请求，所以Redis决定将AOF重写程序放在子进程当中。这样可以同时做到两个不同的目的：

- 子进程进行AOF重写期间，服务器进程（父进程）可以继续处理命令请求。
- 子进程带有服务器进程的数据副本，使用子进程而不是线程，可以避免使用锁的情况下，保证数据的安全性。

但是这还有一个问题，就是重写过程中，服务器接受到了新的命令，如果不写入新的AOF文件当中，会产生不一致的问题。

为了解决这种数据不一致的问题，Redis服务器设置了一个AOF重写缓冲区。

当Redis服务器执行完成一个写命令之后会把这个命令发送给AOF缓冲区和AOF重写缓冲区。

## 重点回顾

- AOF文件通过保存所有修改数据库的写命令请求来记录服务器的数据库状态。
- AOF文件中的所有命令都以Redis命令请求协议的格式保存。
- 命令请求会先保存到AOF缓冲区里面，之后再定期写入并同步到AOF文件。
- appendfsync选项的不同值对AOF持久化功能的安全性以及Redis服务器的性能有很大影响。
- 服务器只要载入并重新执行保存在AOF文件中的命令，就可以还原数据库本来的状态。
- AOF重写可以产生一个新的AOF文件中的命令，这个新的AOF文件的和原来相比保存的数据库状态一样，但体积更小。
- 在执行BGREWRITEAOF命令时，Redis服务器会维护一个AOF重写缓冲区，该缓冲区会在子进程创建新的AOF文件期间，记录服务器执行的所有写命令，当子进程创建新AOF文件的工作之后，服务器会将重写缓冲区中的所有内容追加到新AOF文件的末尾，使得新旧两个AOF文件所保存的数据库状态一致。

