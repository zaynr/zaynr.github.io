---
title: 奇技淫巧-重载malloc抓内存泄漏
date: 2021-01-05 16:31:10
tags:
---

讲到 memory leak，valgrind 是一个不可能绕过的话题。但是在实际应用中，反而 valgrind 能发现的 leak 基本没有。原因其实挺简单的，valgrind walk heap 的时候，他是不知道哪些是需要一直持有的内存区，哪些才是泄漏的内存区。并且丫的得 attach 上去。

所以才需要一个重载的 malloc、calloc、mmap、free 来记录我们每一次内存分配、释放，由开发者自己来 review 可疑的 leak 点而不是由通用检查程序来替你发现。之前在讲这个重载的时候，有小朋友跟我杠过「你这个东西不能用在 C++，new 和 delete 你没重载到。」。这里就多说一句吧：new 和 delete 虽然经常在面试里被提到，说和 malloc、free 是有区别的，但是其实 malloc 就是 new 的一个部分，free 和 delete 的关系同理。

接下来，让我们一步步的去实现这个有趣的 hook。

[内存检测源码](https://github.com/zaynr/mck)

## LD_PRELOAD trick

这个是一切的基础，所以这个东西才只能在 linux 上实现。LD_PRELOAD 可以让用户在启动进程时指定首先读取的 library，包括 glibc。写过 C/C++ 的话，应该会碰到库冲突的问题，多个动态库里如果有同名的 symbol，进程就会使用先读到的那个动态库里的。这也是 LD_LIBRARY_PATH 折磨人的原因之一。

在我们使用了 LD_PRELOAD 成功重载了 glibc 之后，就可以在自定义的那些 malloc、free 等内存操作中开始记录。

```cpp
struct MY_HEAD {
  size_t magic;
  void* realptr;
  size_t size;
};

#define MY_MAGIC_ALLOC 0x55aa66bb77cc88dd // 已分配内存魔术字
#define MY_MAGIC_FREE 0xdd88cc77bb66aa55  // 已释放内存魔数字

extern "C" void* malloc(size_t size) {
    auto rs = sizeof(MY_HEAD) + size;
    // tmalloc: true malloc
    auto p = (MY_HEAD *)tmalloc(rs);
    p->magic = MY_MAGIC_ALLOC;
    p->realptr = p;
    p->size = rs;

    LOG("alloced [%u] byte", size);

    // offset of customized head
    return (p + 1);
}
```

看到以上代码自然很好理解，我们只是在真正的 malloc 等内存操作外面包了一层皮。
但是看到这里，肯定已经有一个疑问：在我们重载了 malloc 之后，真的 malloc 怎么办？


## 获取真正的 malloc

linux 的系统函数 dlsym 里有一个 RTLD_NEXT 选项，可以让调用者在存在符号冲突时直接获取到下一个符号（也就是函数）的指针。所以我们的代码现在应该是这样。

```cpp

bool inited = false; // consider atomic

void __attribute__((constructor)) init() {
    if (inited) {
        return;
    }
    inited = true;

    void* (*tmp_malloc)(size_t size);
    *(void**)(&tmp_malloc)   = dlsym(RTLD_NEXT, "malloc");

    if (!tmp_malloc) {
        frintf(stderr, "load function fail %s\n", dlerror());
        exit(-1);
    }

    tmalloc = tmp_malloc;
}

extern "C" void* malloc(size_t size) {
    if (!inited) {
        init();
    }
    // ...
}
```

但是，如果我们开始运行这段代码的话，我们会发现一个问题：程序崩溃了。

## 中间的混沌

虽然在一开始，我们已经把 malloc 给 overload 掉了，但是我们实际的内存分配还是需要通过 dlsym 获取到 glibc 里真正的 malloc 等内存操作函数指针。

而真正的 malloc(aka tmalloc, true-malloc)，和我们自定义的 malloc(aka cmalloc, customized-malloc) 中间，有一段还没 dlsym 的真空期。

1. LD_PRELOAD 重载 malloc。
2. **malloc 真空期**。 <-- we are here
3. dlsym 真正的 malloc。**dlsym 内的 malloc 当前是 cmalloc！**
4. 将 tmalloc 放进 cmalloc 中。

问题就出在 dlsym 和 printf 等函数，内部也是包含了 malloc 的，也就是说掉进了先有蛋还是现有鸡的陷阱里。在 dlsym、printf、fprintf 等函数内调用到的 malloc 是 cmalloc，而 cmalloc 里又包含了 tmalloc 的调用。在这个真空期里，我们无法获取到 tmalloc 的指针，进而导致了程序的崩溃。

问题已经抛出来了，那么我们有解决办法吗？当然是有解决办法的：用 stack 代替 heap。

众所周知，虽然对内存区分了 stack、heap 等等，但是其实都是同一块内存条上的东西在那边写写画画，区别只是 stack 的容量远小于 heap。那么，tmalloc 拿到的 heap 区内存指针其实也可以用 stack 区的内存指针来代替。这就是我们如何填补上这个无法获取到 heap 区内存指针的真空区的办法：使用一个 stack 上的假 malloc(aka smalloc, stack-malloc)。

```cpp
#define ALIGN(x) ((x + (sizeof(size_t) - 1)) & ~(sizeof(size_t) - 1))
#define MY_HEAP_SIZE (2 * 1024 * 1024)
char my_heap[MY_HEAP_SIZE];

bool inner_call = true; // 混沌期标识

void* smalloc(size_t size) {
  size_t alloc_size = ALIGN(size) + sizeof(size_t);
  if (MY_HEAP_SIZE - my_heap_pos >= alloc_size) {
    char* p     = my_heap + my_heap_pos;
    *(size_t*)p = size;
    p += sizeof(size_t);
    my_heap_pos += alloc_size;

    return p;
  }
  return NULL;
}

void __attribute__((constructor)) init() {
    if (inited) {
        return;
    }

    void* (*tmp_malloc)(size_t size);

    // 混沌期开始
    inner_call = true;
    *(void**)(&tmp_malloc)   = dlsym(RTLD_NEXT, "malloc");

    if (!tmp_malloc) {
        frintf(stderr, "load function fail %s\n", dlerror());
        exit(-1);
    }

    tmalloc = tmp_malloc;
    // 混沌期结束
    inner_call = false;

    inited = true;
}

extern "C" void* malloc(size_t size) {
    // init, etc
    if (inner_call) {
        // memory space for dlsym, printf etc
        return smalloc(size);
    }
    // 下面的调用需要是栈区 smalloc，否则会递归 cmalloc
    inner_call = true;
    // do customized things
    // ...

    // 混沌结束
    inner_call = false;
    return (p + 1);
}
```

## 多线程

经过上面一通操作以后，我们已经把 malloc hook 进了程序里，并且可以愉快的跑起来。

```bash
export LD_PRELOAD=./libmck.so
ls
```

但是现在的服务端进程，是不可能只有单线程的。当多线程被引入，很快我们就发现程序还是会崩溃，stack overflow。问题出在 inner_call 是个全局变量，不是每个线程独占的，所以可能某个线程 inner_call 结束了，但是别的线程还在继续，就导致了递归的 cmalloc 情况出现。

既然是每个线程需要有自己独立的状态，自然就想到了 ```__thread``` 这个修饰符。其修饰的变量在每个线程都有自己独立的一份空间，不会有互相冲突的情况。

```cpp
__thread bool inner_call = true;
```

## 内存占用排序

解决掉各种各样的问题之后，终于到了最后一步：统计内存使用情况。对于这个部分的实现倒不用很详细的讲，各花入各眼，想怎么实现就怎么实现了。在作为样例的 project 里，是维护了一个最大堆，把所有的内存占用依据大小给存进来。

## backtrace 之殇

对于调用栈的记录，目前使用的是简单粗暴的 backtrace，也就是每次 malloc/free 都会调用一次 backtrace。明眼人一看就出来了：这不是 cpu 会飙的飞起吗？

是的，使用这个方法会极大影响到程序性能。

所以考虑其他办法则成为了这个工具的下一个问题。

## 原子操作

待续。
