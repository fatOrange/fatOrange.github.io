---
layout: article
title:  存储模型
tags: [C++,mutli_thread,memory_model]
author: fatOrange
aside:
  toc: true
sidebar:
  nav: multithread
---

# 多线程内存模型

**一个基本原理**:原子变量主要是用来同步不同线程间对内存的访问

**存储模型的作用**:指定不同线程之间数据访问的关系

**Atomic类的主要操作方法**:load()，store()

## 原子操作的内存顺序

这里有六个内存序列选项可应用于对原子类型的操作：

| 内存序列选项                           | 内存序列模型（同步和排序约束）        | 原子操作                                                    | 关联操作            |
| -------------------------------------- | ------------------------------------- | ----------------------------------------------------------- | ------------------- |
| memory_order_relaxed                   | 松散序列(memory_order_relaxed)        | 读写                                                        |                     |
| ~~memory_order_consume~~(暂不推荐使用) | 获取-释放序列                         | 读                                                          |                     |
| memory_order_acquire                   | 获取-释放序列                         | 读（对后面有要求  不能早于我 后面要用到这个值，对读最有效） | Load                |
| memory_order_release                   | 获取-释放序列                         | 写（对上面有要求  不能晚于我，对写最有效）                  | store               |
| memory_order_acq_rel                   | 获取-释放序列                         | 读写                                                        | A read-modify-write |
| memory_order_seq_cst(默认)             | 排序一致序列(sequentially consistent) | 读写                                                        | A read-modify-write |

不同的内存序列模型在不同的CPU架构下，功能是不一样的。

## 详细说明

### memory_order_relaxed  

Relaxed operation: there are  no synchronization or ordering constraints  imposed on other reads or writes, only this operation's atomicity is  guaranteed(see Relaxed ordering below)   [![dkumeH.png](https://s1.ax1x.com/2020/08/15/dkumeH.png)](https://imgchr.com/i/dkumeH)

------

没有对其他读取或写入施加任何同步或排序约束，仅保证此操作的原子性



### ~~memory_order_consume~~ 暂不使用

  A load operation with this  memory order performs a *consume  operation* on  the affected memory location: no reads or writes in the current thread  dependent on the value currently loaded can be reordered before this load.  Writes to data-dependent variables in other threads that release the same  atomic variable are visible in the current thread. On most platforms, this  affects compiler optimizations only![dkuuTA.png](https://s1.ax1x.com/2020/08/15/dkuuTA.png)

------

具有此内存顺序的装入操作在受影响的内存位置上执行消耗操作：在此装入之前，不能根据当前装入的值对当前线程中的任何读取或写入进行重新排序。 在释放相同原子变量的其他线程中写入与数据相关的变量在当前线程中可见。 在大多数平台上，这仅影响编译器优化。



### memory_order_acquire  

A load operation with this memory order performs the *acquire operation* on the affected memory location: no reads or writes in the current thread can be reordered before this load. All writes in other threads that release the same atomic variable are visible in the current thread。

[![dkunwd.png](https://s1.ax1x.com/2020/08/15/dkunwd.png)](https://imgchr.com/i/dkunwd)

------

带有此内存顺序的加载操作在受影响的内存位置上执行获取操作：在此加载之前，无法对当前线程中的任何读取或写入进行重新排序。 其他线程中释放相同原子变量的所有写操作在当前线程中可见 。



###  memory_order_release  

A store operation with this memory order performs the *release operation*: no reads or writes in the current thread can be reordered after this store. All writes in the current thread are visible in other threads that acquire the same atomic variable (see Release-Acquire ordering below) and writes that carry a dependency into the atomic variable become visible in other threads that consume the same atomic(see Release-Consume ordering below).

[![dkuMFI.png](https://s1.ax1x.com/2020/08/15/dkuMFI.png)](https://imgchr.com/i/dkuMFI)

------

具有此内存顺序的存储操作执行释放操作：在此存储之后，无法对当前线程中的任何读取或写入进行重新排序。 当前线程中的所有写操作在获取相同原子变量的其他线程中可见，而将依赖项带入原子变量的写操作在使用相同原子的其他线程中可见



###  memory_order_acq_rel  

A read-modify-write operation with this memory order is both an *acquire operation* and a *release operation*. No memory reads or writes in the current thread can be reordered before or after this store. All writes in other threads that release the same atomic variable are visible before the modification and the modification is visible in other threads that acquire the same atomic variable.

[![dkuQYt.png](https://s1.ax1x.com/2020/08/15/dkuQYt.png)](https://imgchr.com/i/dkuQYt)

------

具有该存储顺序的读-修改-写操作既是获取操作又是释放操作。 在此存储之前或之后，无法对当前线程中的任何内存读写进行重新排序。 修改之前，其他线程中释放相同原子变量的所有写操作都是可见的，而修改在其他获得相同原子变量的线程中可见。



###  memory_order_seq_cst  

A load operation with this memory order performs an *acquire operation*, a store performs a *release operation*, and read-modify-write performs both an *acquire operation* and a *release operation*, plus a single total order exists in which all threads observe all modifications in the same order(see Sequentially-consistent ordering below) 

[![dkulfP.png](https://s1.ax1x.com/2020/08/15/dkulfP.png)](https://imgchr.co

------

以该内存顺序执行的装入操作执行获取操作，存储执行释放操作，读-修改-写操作同时执行获取操作和释放操作，此外还存在一个总顺序，在该顺序中，所有线程均会观察内存中的所有修改。 相同的顺序

## Quiz

```cpp
 -Thread 1-       			-Thread 2-                   	-Thread 3-
 y.store (20);    			if (x.load() == 10) {        	if (y.load() == 10)
 x.store (10);      			assert (y.load() == 20)      	assert (x.load() == 10)
                    			y.store (10)
                  			}

```

Q：请说明 sequentially consistent，Release，acquire 这三种mode在同时运行这段线程时有什么区别？

A：

如果使用sequentially consistent mode 线程2和3的 accert 全部 pass 因为他是顺序一致的，这三个线程都存在同步关系

如果使用 Release/acquire mode  线程2 pass 线程3 不pass，因为线程1和线程2的 **Release/acquire 可以成对使用，因此这两个线程存在同步关系，而线程1和线程3没有成对使用所以不存在同步关系**

## 引用

https://chenxiaowei.gitbook.io/concurrency-with-modern-c/xiang-xi-jie-shao/1.0-chinese/1.4-chinese

https://chenxiaowei.gitbook.io/c-concurrency-in-action-second-edition-2019/5.0-chinese/5.3-chinese#5-3-5-zha-lan

https://en.cppreference.com/w/cpp/atomic/

https://gcc.gnu.org/wiki/Atomic/GCCMM/AtomicSync