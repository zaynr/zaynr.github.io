---
title: C++ POD 内存操作
date: 2020-02-24 22:18:35
tags:
---

C++ 在涉及到网络编程情况下，一般操作是在一个 header 里带上 attachment 的长度，再分配一个对应长度的 buffer 接收 attachment。但是一些情况下，无法预先分配一个恰到好处的 buffer 来接收数据，比如使用 curl 时。所以需要对不同的 buffer 操作方式进行比较，从而得出最佳操作的方式。

以下 benchmark 是对一个 2mb 的内存块操作。

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

## memset vs std::fill

memset 是 asm 内嵌的操作，而 std::fill 是一个模版壳子的 for 循环，所以性能差距会很大。但我没想到会这么大。


|           | memset | std::fill |
| --------- | ------ | --------- |
| 1000 次   | 341ms  | 17323ms    |

## memcpy vs std::copy

std::copy 的内部实现其实就是 memcpy/memmove, 所以性能差距不会很大，甚至 std::copy 会表现比 memcpy 更好。



|           | memcpy | std::copy |
| --------- | ------ | --------- |
| 100000 次 | 1184ms | 1175ms    |


## 实际操作

1. vector reserve copy
    - reserve 一个 2mb 的 vector 作为 buffer
    - copy 2mb 的内存块到 vector 内
    - 析构

2. vector resize copy
    - resize 一个 2mb 的 vector 作为 buffer
    - copy 2mb 的内存块到 vector 内
    - 析构

3. 内存块
    - malloc 一个 2mb 的 char* 作为 buffer
    - copy 2mb 的内存块到 char* 内
    - free

4. vector insert
    - reserve 一个 2mb 的 vector 作为 buffer
    - insert 2mb 的内存块到 vector 内
    - 析构

|       | vector reserve copy | vector resize copy | 内存块 | vector insert |
| ----- | ------------------- | ------------------ | ------ | ------------- |
| 100次 | 337ms               | 21972ms            | 400ms  | 10594ms       |

### 使用 vector 的 resize、copy 来向后 append 数据。

这种办法是效率最低，但是也是（内存管理）最安全的方法。由 vector 来管理分配的堆空间。影响主要是 resize 会调用 std::fill，导致性能极差。

### 使用 vector 的 insert 来补充进入的数据。

这种办法看似比 resize 效率高，但是涉及到 capacity 不足导致 vector 重新分配堆空间、拷贝原先数据的情况。可以使用 reserve 一个较大空间来避免这种情况，但是还是无法完全避免。insert 也类似于 fill，是一个循环。

### 使用 char* malloc 内存数据。

这种办法是理论上最快的办法，可以使用 unique_ptr 来避免内存泄漏。虽然过程中会有重复的 malloc、memcpy，但是 memcpy 是 asm 操作，比套一层模板外壳的 for 循环性能高上许多。malloc 也可参考 vector 的实现，每次重新 malloc 都是 2*capability。

### 使用 char* realloc 内存数据。

这个办法是 curl 官方网站提供的。但是一般情况下 realloc 是不安全的，因为realloc 后的指针**可能**指向一个新的堆。

1. 旧的内存区无法清空，可能会有敏感数据。
2. 会有人写的比较挫，类似 ```p = realloc(p, new_size);```。如果分配失败会导致 p 直接丢失。

其实 realloc 就是做了上面 malloc、memcpy 的事情，但是丢失了一定的自由度。当然，curl 官方这么写，是安全的。但是一般情况下，realloc 应该作为最后的一个选项出现。
