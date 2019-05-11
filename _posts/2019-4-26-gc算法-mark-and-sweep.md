---
layout: post
title:  "gc算法-mark-and-sweep"
date:   2019-4-26 22:00:00
categories: 基础知识
---


我是按自己的理解总结了下[这篇文章](https://www.geeksforgeeks.org/mark-and-sweep-garbage-collection-algorithm/)

所有的垃圾回收算法都有两个基本操作,第一个是找到所有未被使用的object,第二个是让被回收的堆空间可以再次被使用。

在**Mark and Sweep 算法**中,这两个操作

分为两部分:

- Mark阶段
- Sweep阶段

**Mark阶段**

当一个object被创建的时候,它的mark bit默认为0。

在Mark阶段,会做一个深度优先遍历,将所有被涉及到的Obejct的mark bit置为1。

```
Mark(root)
    If markedBit(root) = false then
        markedBit(root) = true
        For each v referenced by root
             Mark(v)
```

**Sweep阶段**

在Sweep阶段,遍历堆中的全部object,将mark bit为1的全部置为0,将为0的release掉。

```
Sweep()
For each object p in heap
    If markedBit(p) = true then
        markedBit(p) = false
    else
        heap.release(p)
```

然后在跑一次Mark()

Mark And Sweep算法的优点

- 不会引起无限循环
- 在执行算法的时候不会有额外开销

Mark And Sweep算法的缺点

- 在执行算法的时候,程序会被挂起。
- 会产生很多的内存碎片。