---
layout: post
title:  "mysql-buffer pool"
date:   2019-4-29 22:00:00
categories: 数据库
---

# 缓冲池

实际上是一块内存区域,存储的是一些表数据和索引数据。当查询的时候先去缓冲池里面查,如果没有再去磁盘上查。
通过这种方式可以极大的提升查询速度。

缓冲池中存储的不是一条条数据,而是一页数据。每一页都是列表中的一条数据。

> page:作为磁盘和内存交换数据的单位。包含了一行或多行数据，具体是多少依赖于一行有多少数据。

缓冲池使用的算法是变种的LRU算法。插入的位置不是头部,而是大概5/8的位置,称此位置为midpoint。
插入的时候先将缓冲池中最近最少使用的一页删掉,然后将新的一页插入到midpoint。

![avatar](https://raw.githubusercontent.com/daysleep666/blog/master/src/img/article/bufferpool.jpeg)

当数据被插入到缓冲池中的时候,不插入头部而是插入到midpoint位置的目的是避免真正的热点数据被从缓冲池中删掉。
> 如果插入到缓冲池头部，当Select整个表的时候会将那些并不经常使用的数据插入到缓冲池中，将真正的热点数据挤出缓存池。

当读取的那一页数据在old sublist中的时候,将其移动到new sublist。

--------

CheckPoint

将脏页刷新回硬盘

刷新策略：

1. 缓冲池满了
2. 定时