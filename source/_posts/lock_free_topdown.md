---
title: lock free - 自顶向下
date: 2020-02-05 16:08:01
tags:
---

lock free 作为性能控们眼中的弄潮儿技术，自然是有着让人难以抗拒的性能魅力。

但是在讨论 lock free 之前，我们先来大概看看 lock 吧。毕竟没有对比，就没有对性能的概念。

## spin lock

spin lock 会在一个死循环内反复尝试，直到拿到锁为止。
优点：如果获取不到锁，也不会陷入内核态。不会响应中断。
缺点：CPU 占用极高。
这种锁一般用在 log 模块内，向 log buffer 写时使用 spin lock。

## semaphore

信号量的实现可以认为是一种带了引用计数的锁，内部应用了 spin lock 来保护引用计数。

## mutex lock

最常见的锁，也是最应该被用到的锁。可以看作是引用计数为 1 的 semaphore。如果获取不到锁，会触发软中断，陷入内核态，让出 CPU，让内核来调度下一个时间片。使用这种锁最坏情况是某个线程死锁，是一种万金油锁，大部分场景用它都不能说是错的。

## recursive lock

mutex lock 的可重入版本。如果同一个线程对同一个 mutex 反复加锁，不会导致死锁。在解锁时也需要和加锁次数配对的解锁次数。这种锁在应用上一般是写的很挫的东西，实在用 mutex 改不动了才会加一个 recursive。

## rwlock

读写锁，实现为加了 read count 和 write count。可以把 read lock 当成多线程方面的 recursive lock，write lock 方面的 mutex lock。一般来说适用于配置方面的保护。

## 性能比较

```s
硬件概览：
  型号名称：	MacBook Pro
  型号标识符：	MacBookPro15,4
  处理器名称：	四核Intel Core i5
  处理器速度：	1.4 GHz
  处理器数目：	1
  核总数：	4
  L2缓存（每个核）：	256 KB
  L3缓存：	6 MB
  超线程技术：	已启用
  内存：	8 GB
  系统固件版本：	1554.80.3.0.0 (iBridge: 18.16.14347.0.0,0)
  激活锁状态：	已启用
```

使用以上介绍的几种锁，开启 4 个线程，对同一个 stack 进行 push、pop 操作，获取其吞吐量。

### 1 push, 3 pop

在使用 rwlock 时，对这种场景的性能测试没有什么意义。

## atomic

## memory barrier

## CAS
