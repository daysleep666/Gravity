---
layout: post
title:  "snowflake的简单实现"
date:   2018-11-18 17:45:00
categories: 算法
---

在分布式开发中，如何生成id是一个很重要的事情，有两个要求，一个是有序，另一个是唯一。
有序是因为在数据库中查找有序数据会更快一些，唯一是为了避免并发的问题。


snowflake生成分布式id算法

```
	snowflake共64位
	第一位 占用1bit，始终为0。在二进制中，第一位代表正数和负数，我们生成的id一般为正数，所以保持为0.
	第二位 毫秒级别的时间戳，占41bit
	第三位 工作机器id，占10bit，最大可以同时有1024个工作进程。(10bit可以细分为高5bit的数据中心id，低5bit的工作节点id)
	第四位 序列号 由0递增到4096 占12位，可以保证同一毫秒内可以生成4096个id。
	理论上可以在同一毫秒内生成1024*4096=4194304个id。		
	同一毫秒同一进程可以产生4096个id。
```

```

var INITTIMESTAMP uint64
var index, WORKID uint64
var lastTimeStamp uint64
var mt sync.Mutex

func init() {
	INITTIMESTAMP = uint64(time.Date(2018, 1, 1, 0, 0, 0, 0, time.Now().Location()).UnixNano())
	WORKID = 1
}

func main() {
	var wg sync.WaitGroup
	var count int64 = 10000
	wg.Add(int(count))
	for i := int64(0); i < count; i++ {
		go func() {
			fmt.Println(snowFlake(), index)
			wg.Done()
		}()
	}
	wg.Wait()
}

func snowFlake() uint64 {
	mt.Lock()
	defer mt.Unlock()

	timeNow := uint64(time.Now().UnixNano())
	if lastTimeStamp == timeNow {
		index = (index + 1) & 111111111111
		if index == 0 {
			for lastTimeStamp == timeNow {
				timeNow = uint64(time.Now().UnixNano())
			}

		}
	} else {
		index = 0
	}
	lastTimeStamp = timeNow
	return ((timeNow-INITTIMESTAMP)<<22 | WORKID<<12 | index)
}

```