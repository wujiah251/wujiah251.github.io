title: C++多线程编程相关操作
author: Jiahao Wu
tags: []
categories:
  - 操作系统
date: 2021-02-17 20:16:00
---
前段时间写了个轻量级的多线程Web服务器，今天顺便梳理一下相关的操作。  

# 线程

```C++
int pthread_create(pthread_t *tid,const const pthread_attr_t *attr,void *(*func)(void*),void *arg);
// 上述函数可以创建一个线程，如果创建成功，线程ID存入tid
// attr用于设置线程属性，优先级、栈大小等等
// func、arg是线程执行的函数及参数
// 创建成功返回0，否则非0
int pthread_join(pthread *tid,void **status);
// 等待一个线程结束，tid指针存储这个线程的id
// 如果status非空，用于存储这个线程的返回值
ptherad_t pthread_self(void);
// 返回自身线程的id
int pthread_detach(pthread_t tid);
// 将一个线程转变为脱离状态，即终止时就释放所有资源
// 另一个状态时joinable，这个状态下终止后将会等待其他线程对本线程调用pthread_join
// 此前将会保存ID和退出状态
void pthread_exit(void *status);
// 终止线程
```

# 线程之间的同步技术

## 信号量sem

信号量类型是``sem_t``：  
```C++
int sem_init(sem_t* sem,int pshared,unsigned int value);
// 初始化一个信号量
// pshared表示是否在进程间共享，0表示只在线程间共享，否则进程间共享
// value为设置的初始值
int sem_destroy(sem_t* sem);
// 销毁一个线程
int sem_wait(sem_t* sem);
// P操作，对信号量-1
int sem_post(sem_t* sem);
// V操作，信号量+1
int sem_getvalue(sem_t* sem, int* valp);
// 返回信号量的值到valp
```

## 互斥锁mux

互斥锁：
```C++
int pthread_mutex_init(pthread_mutex_t* mutex, const thread_mutexattr_t* mutexattr);
// 初始化一个互斥锁，mutexattr是相关设置参数
int pthread_mutex_lock(pthread_mutex_t* mutex);
// 对互斥锁加锁
int pthread_mutex_unlock(pthread_mutex_t* mutex);
// 解锁
int pthread_mutex_trylock(pthread_mutex_t* mutex;
// 非阻塞加锁，如果已经上锁，不会阻塞，避免死锁
int pthread_mutex_destroy(pthread_mutex_t* mutex);
// 用来撤销互斥锁的资源。
```

## 读写锁

读写锁是用来处理读者-写者问题：  
```C++
int pthread_rwlock_init(pthread_rwlock_t* rwlock, const pthread_rwlockattr_t* attr);
// 初始化读写锁
int pthread_destroy(pthread_rwlock_t* rwlock);
// 销毁读写锁
int pthread_rwlock_rdlock(pthread_rwlock_t* rwlock);
// 加读锁
int pthread_rwlock_wrlock(pthread_rwlock_t* rwlock);
// 加写锁
int pthread_rwlock_unlock(pthread_rwlock_t* rwlock);
// 解锁
```

## 自旋锁

自旋锁和互斥锁差不多，区别是自旋锁阻塞方式和互斥锁不同，互斥锁是让线程睡眠来实现阻塞，而自旋锁是通过不断循环让线程忙等待，适用于占用自旋锁时间比较短的情况。
```C++
int pthread_spin_init(__pthread_spinlock_t* __lock, int__pshared);
int pthread_spin_destroy(__pthread_spinlock_t* __lock);
int pthread_spin_trylock(__pthread_spinlock_t* __lock);
int pthread_spin_unlock(__pthread_spinlock_t* __lock);
int pthread_spin_lock(__pthread_spinlock_t* __lock);
```

# 






