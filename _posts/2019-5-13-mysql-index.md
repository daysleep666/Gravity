---
layout: post
title:  "Innodb-index整理01"
date:   2019-4-29 22:00:00
categories: 数据库
---

# Innodb-索引

### 主键索引

1. 一个表只能有一个主键索引 
2. 主键索引树里放着的是全部的数据
3. 不允许重复
4. 不允许为NULL

- 创建索引

```
    ALTER TABLE tableName ADD PRIMARY KEY(columnName);
```

```
    CREATE TABLE tableName (id INT NOT NULL, PRIMARY KEY (id));
```

- 删除索引
```
    ALTER TABLE tableName DROP PRIMARY KEY;
```

### 次级索引

1. 一个表可以有多个次级索引
2. 索引树里放着的是主键和索引数据

#### 普通索引

1. 可以重复
2. 可以为NULL

- 创建索引
```
    CREATE INDEX indexName ON tableName(columnName);
```

```
    ALTER TABLE tableName ADD INDEX indexName(columnName);
```

```
    CREATE TABLE tableName (
        id INT NOT NULL,
        INDEX indexName (columnName)
    );
```

#### 唯一索引

1. 不可以重复
2. 可以为NULL，但是这时候就可以插入多个NULL值了。

- 创建索引
```
    CREATE UNIQUE INDEX indexName ON tableName(columnName);
```

```
    ALTER TABLE tableName ADD UNIQUE indexName (columnName);
```

```
    CREATE TABLE tableName(
        id INT NOT NULL,
        UNIQUE indexName (columnName)
    )
```

- 删除索引
```
    DROP INDEX indexName ON tableName;
```

```
    ALTER TABLE tableName DROP indexName;
```

- 查询索引
```
    SHOW INDEX FROM tableName;
````

----

聚簇索引

按照聚簇索引索引的顺序存放在硬盘上

聚簇索引的构建

1. 如果存在主键索引就用主键索引作为聚簇索引
2. 如果不存在主键索引就用一个非空唯一索引做为聚簇索引
3. 如果上面两种都不满足，就创建一个虚拟的聚簇索引

