---
layout: post
title:  "MVCC"
date:   2019-5-19 22:00:00
categories: 数据库
---

MVCC

    多版本数据控制，无需加锁就可以实现并发读

创建时间和删除时间

    并不是真正意义上的时间戳，而是事务id。

undolog

    逻辑日志，记录相反的操作，用来回滚和找到旧版本数据。

SELECT

    每条数据都有一个创建时间和删除时间，事务在读数据的时候会读那些创建时间小于等于自己且删除时间大于自己或者未定义的数据。

INSERT

    新数据的创建时间是系统当前事务id，删除时间未定义。
    向undolog里插入一条delete语句。

UPDATE

    将该数据的删除时间修改为当前事务id，插入一条修改后的数据，创建时间为当前事务id。
    向undolog里插入一条相反的update语句。

DELETE

    插入一条修改后的数据，将原数据的删除时间修改为当前事务id。
    向undolog里插入一条insert语句。

<!-- 先写undolog在写redolog -->
<!-- https://www.cnblogs.com/f-ck-need-u/archive/2018/05/08/9010872.html#auto_id_12 -->
<!-- https://blog.csdn.net/w2064004678/article/details/83012387 -->
<!-- https://www.cnblogs.com/myseries/p/10930910.html  写的很好-->