---
title: 常见线程锁介绍（C++）
description: 互斥锁、条件锁、自旋锁、读写锁、递归锁的应用
slug: threadLockcpp
date: 2024-05-24 00:00:00+0000
categories:
    - 线程锁
    - C++
tags:
    - 文档
weight: 1       # You can add weight to some posts to override the default sorting (date descending)
params:
    author: 王新晓
---

# 常见线程锁介绍（C++）
互斥锁、条件锁（条件变量）、自旋锁、读写锁、递归锁的应用

## 互斥锁
在多线程情况下，不同线程争对同一份临界资源进行操作时使用的锁，保证临界资源只有一个线程使用，C++ 11 中引入了std:mutex ，linux平台C函数中也有方法

加锁后需要解锁，其他线程才能进入，否则会一直等待

### C函数（Linux）

```c++
// 声明一个互斥量    
pthread_mutex_t mtx;
// 初始化 
pthread_mutex_init(&mtx, NULL);
// 加锁  
pthread_mutex_lock(&mtx);
// 解锁
pthread_mutex_unlock(&mtx);
// 销毁
pthread_mutex_destroy(&mtx);
// 尝试加锁，加锁成功返回0
pthread_mutex_trylock(&mtx);
```

### C11 (全平台)

```c++
// 声明一个互斥量（std::mutex不允许拷贝构造，也不允许 move 拷贝）
std::mutex mtx;
// 加锁  
mtx.lock();
// 解锁
mtx.unlock();
// 尝试加锁，加锁成功返回true
mtx.try_lock();

//RAII应用，使用unique_lock和lock_guard通过构造和析构自动加锁解锁
std::unique_lock<std::mutex> locker(mtx);
std::lock_guard<std::mutex> locker(mtx);
//unique_lock可以手动加锁解锁，lock_guard只能通过构造析构加锁解锁
```



## 条件锁（条件变量）

多线程中，未满足条件的线程会阻塞住，直到条件满足将继续执行的情况下使用的锁

### C函数（Linux)

```c
// 申明一个互斥量
pthread_cond_t cond;
// 初始化
pthread_cond_init(&cond);
// 销毁
pthread_cond_destroy(&cond);
/** 阻塞等待一个条件变量
* 这里将执行三个操作：
* 	1. 阻塞并等待条件变量cond满足
* 	2. 释放已锁住的互斥锁mtx，（相当于pthread_mutex_unlock(&mtx);）
*	3. 被唤醒后，pthread_cond_wait返回时，解除阻塞并锁住mtx（相当于pthread_mutex_nlock(&mtx);）
* 为什么需要搭配互斥锁mtx使用将在后面解释
*/
pthread_cond_wait(&cond, &mtx);

struct timespec {
	time_t tv_sec; // seconds  秒
	long tv_nsec;  // nanosecondes 纳秒 
}
timespec abstime;
/** 限时阻塞等待一个条件变量
* 除了执行pthread_cond_wait(&cond, &mtx);的操作
* 还会在abstime时间内等待
* 需要注意的是，abstime要填的是1970年1月1日以来到想要停止的时间的秒数
* 超时将返回ETIMEDOUT（非0）
*/
pthread_cond_timedwait(&cond, &mtx， &abstime);
// 唤醒至少一个阻塞在条件变量上的线程
pthread_cond_signal(&cond);
// 唤醒全部阻塞在条件变量上的线程
pthread_cond_broadcast(&cond);
```

### C11（全平台）

```c++
// 申明一个条件变量
std::condition_variable cv;
std::lock_guard<std::mutex> locker(mtx);
//这里将要使用互斥锁中的unique_lock

/** 阻塞等待一个条件变量
* 这里将执行三个操作：
* 	1. 阻塞并等待条件变量cv满足
* 	2. 释放已锁住的互斥锁mtx，（相当于locker.unlock()）
*	3. 被唤醒后，wait返回时，解除阻塞并锁住mtx（相当于locker.lock()）
* 为什么需要搭配互斥锁mtx使用将在后面解释
*/
cv.wait(locker);
// 限时阻塞等待一个条件变量，知道经过多久时间，超时将返回false
cv.wait_for(locker, rel_time);
// 限时阻塞等待一个条件变量，直到等到某个时间点，超时将返回false
cv.wait_unitl(locker, point_time);
// 唤醒一个在cv上等待的线程
cv.notify_one();
// 唤醒任何在cv上等待的线程
cv.notify_all()
```



> [!NOTE]
>
> 为什么条件锁（条件变量）需要搭配互斥锁使用？
>
> 可能会存在永久阻塞的情况

