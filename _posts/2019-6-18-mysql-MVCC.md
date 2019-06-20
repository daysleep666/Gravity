---
layout: post
title:  "MVCC"
date:   2019-6-18 22:00:00
categories: 数据库
---

MVCC

    多版本数据控制，非锁定一致性读

创建时间和删除时间

    并不是真正意义上的时间戳，而是事务id。

undolog

    逻辑日志，用来回滚和找到旧版本数据。

数据的历史版本

    在Innodb中，数据拥有多个版本，但是只保存最新的版本，历史版本是通过最新版本和undolog推出来的。

每个事务在启动的时候都会有一个保存当前活跃事务id的数组(T_Min,T_Max),

SELECT

    读取规则:
            1.如果数据id<T_Min则可见
            2.如果数据id>=T_Max则不可见
            3.如果数据T_Min<=id&&id<T_Max，则如果id在活跃事务id数组中就不可见，反之可见

INSERT

    新数据的版本是当前事务id
    插入insert_undo，在事务提交后，直接删除undolog

UPDATE

    修改该数据
    插入update_undo，在事务提交后，加入待删除列表，等待purge线程处理

DELETE

    将该数据标记为删除
    插入update_undo，在事务提交后，加入待删除列表，等待purge线程处理

------------------

事务隔离级别RR

假设当前事务id为1000

当前已存在table拥有一条数据

id|data|版本id|删除位
-|-|-|-
1|a|900|0

------

Session A [记录的活跃的事务id是(1000,1001)]

    START TRANSACTION WITH CONSISTENT SNAPSHOT;
    SELECT * FROM table;

id|data|版本id|删除位
-|-|-|-
1|a|1000|0

Session A查到了版本id低于当前事务id的数据  900<1000

---------------------
(当前事务id为1001)

Session B [记录的活跃的事务id是(1000,1001,1002)]

    START TRANSACTION WITH CONSISTENT SNAPSHOT;
    INSERT INTO table VALUES (2,'b');
    SELECT * FROM table;

id|data|版本id|删除位
-|-|-|-
1|a|1000|0
2|b|1001|0

-----------

Session A

    SELECT * FROM table;

id|data|版本id|删除位
-|-|-|-
1|a|1000|0

新插入数据的版本id大于SessionA的事务id，所以对SessionA来说该数据不可见

-------

Session B

    UPDATE table SET data = 'aa' WHERE id = 1;
    SELECT * FROM table;

id|data|版本id|删除位
-|-|-|-
1|aa|1001|0
2|b|1001|0

修改数据，插入一条undolog

-------

Session A

    SELECT * FROM table;

id|data|版本id|删除位
-|-|-|-
1|a|1000|0
2|b|1001|0

Session A查到数据(id=1)的版本大于当前事务id，通过undolog指针找到数据(id=1),版本id=1000的数据