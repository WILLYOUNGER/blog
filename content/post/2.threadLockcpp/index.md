---
title: 常见线程锁介绍（C++）
description: 互斥锁、条件锁（条件变量）、自旋锁、读写锁、递归锁的应用
slug: threadLockcpp
date: 2024-05-25 00:00:00+0000
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

```c
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



> 为什么条件锁（条件变量）需要搭配互斥锁使用（作为参数传入）？
>
> 会出现虚假唤醒的情况，所以需要传入一个locker，防止notify_one（signal）后多个wait同时响应

## 自旋锁

在多线程情况下，不同线程争对同一份临界资源进行操作时使用的锁，保证临界资源只有一个线程使用，**与互斥锁不同的地方在于，如果临界资源被占用，互斥锁会阻塞等待，自旋锁会循环获取锁**

当等待时间较短时可以选择使用自旋锁，可以减少操作系统的用户态与内核态的转换，等待时间较长选择使用互斥锁。

自旋锁在C11中并没有给出实现，需要自己使用std::atomic来进行实现

### C函数（Linux）

```c
// 声明一个互斥量    
pthread_spinlock_t lock;
// 初始化 
pthread_spin_init(&lock);
// 加锁
pthread_spin_lock(&lock);
// 解锁
pthread_spin_unlock(&lock);
//销毁
pthread_spin_destroy(&lock);
// 尝试加锁，加锁成功返回0
pthread_spin_trylock(&lock);
```

### C11(全平台)

这里使用atomic实现了自旋锁

```c++
// 使用C++11的原子操作实现自旋锁（默认内存序，memory_order_seq_cst）
class spin_mutex {
    // flag对象所封装的bool值为false时，说明自旋锁未被线程占有。  
    std::atomic<bool> flag = ATOMIC_VAR_INIT(false);       
public:
    spin_mutex() = default;
    spin_mutex(const spin_mutex&) = delete;
    spin_mutex& operator= (const spin_mutex&) = delete;
    void lock() {
        bool expected = false;
        // CAS原子操作。判断flag对象封装的bool值是否为期望值(false)，若为bool值为false，与期望值相等，说明自旋锁空闲。
        // 此时，flag对象封装的bool值写入true，CAS操作成功，结束循环，即上锁成功。
        // 若bool值为为true，与期望值不相等，说明自旋锁被其它线程占据，即CAS操作不成功。然后，由于while循环一直重试，直到CAS操作成功为止。
        while(!flag.compare_exchange_strong(expected, true)){ 
            expected = false;
        }      
    }
    void unlock() {
        flag.store(false);
    }
};
```

## 读写锁

多线程中对，临界资源的访问方式有两种：读和写。其中，**写操作是独占的**而**读操作是非独占的**，多个线程可以同时读这个共享变量，读写锁就可以用在这种情况下

### C函数（Linux)

```c
// 声明一个读写锁
pthread_rwlock_t  m_rw_lock;
// 初始化
pthread_rwlock_init(&m_rw_lock, NULL);
// 读加锁
pthread_rwlock_rdlock(pthread_rwlock_t*);
// 读尝试加锁，成功返回0
pthread_rwlock_tryrdlock(pthread_rwlock_t*);
// 写加锁
pthread_rwlock_wrlock(pthread_rwlock_t*);
// 写尝试加锁，成功返回0
pthread_rwlock_trywrlock(pthread_rwlock_t*);
// 解锁
pthread_rwlock_unlock(pthread_rwlock_t*);
// 销毁
pthread_rwlock_destroy(pthread_rwlock_t* );
```



### C17（全平台）

```c++
// 声明
std::shared_mutex mtx;

//独占锁定
// 加锁  
mtx.lock();
// 解锁
mtx.unlock();
// 尝试加锁，加锁成功返回true
mtx.try_lock();

//共享锁定
// 加锁  
mtx.lock_shared();
// 解锁
mtx.unlock_shared();
// 尝试加锁，加锁成功返回true
mtx.try_lock_shared();

//RAII应用，使用unique_lock和shared_lock通过构造和析构自动加锁解锁
//shared_lock是在读的时候使用，unique_lock在写的时候使用
std::unique_lock<std::mutex> locker(mtx);
std::shared_lock<std::mutex> locker(mtx);
```



## 递归锁

递归锁可以重复加锁，会记录加锁的次数，每次加锁，计数+1，且判断如果已经锁住，不做处理，每次解锁，计数-1

### C 函数（Linux)

```c
// 声明
pthread_mutexattr_t attr;
// 初始化 
pthread_mutexattr_init(&mtx, NULL);
// 加锁  
pthread_mutexattr_lock(&mtx);
// 解锁
pthread_mutexattr_unlock(&mtx);
// 销毁
pthread_mutexattr_destroy(&mtx);
// 尝试加锁，加锁成功返回0
pthread_mutexattr_trylock(&mtx);
```

### C11（全平台）

```c++
// 声明
std::recursive_mutex rmtx;
// 加锁  
rmtx.lock();
// 解锁
rmtx.unlock();
// 尝试加锁，加锁成功返回true
rmtx.try_lock();

//RAII应用，使用unique_lock和lock_guard通过构造和析构自动加锁解锁
std::unique_lock<std::mutex> locker(rmtx);
std::lock_guard<std::mutex> locker(rmtx);
//unique_lock可以手动加锁解锁，lock_guard只能通过构造析构加锁解锁
```

