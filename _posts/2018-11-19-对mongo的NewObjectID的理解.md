---
layout: post
title:  "mongo的id生成算法"
date:   2018-11-19 17:45:00
categories: 算法 
---

在使用golang的mongo包的时候发现里面也有一个唯一id的生成算法:bson.NewObjectId().

类似snowflake算法，通过主机名的md5，进程id和递增的数来保证id的唯一性。

在同一毫秒内，总共可以生成2^24*2^16*2^24=2^64个id

同一进程，同一毫秒内可以生成2^24个id。

这个函数除了一个原子性操作外是无锁的，所以在同一毫秒内，若一个进程需要产生的id数量大于2^24，那就会
出现id重复的情况。

```
// NewObjectId returns a new unique ObjectId.
func NewObjectId() ObjectId {
	var b [12]byte      // 总共十二字节，96位
	// Timestamp, 4 bytes, big endian
	binary.BigEndian.PutUint32(b[:], uint32(time.Now().Unix())) // 将时间戳秒以大端方式写入里面
	// Machine, first 3 bytes of md5(hostname)  主机名的md5值占24位
	b[4] = machineId[0] 
	b[5] = machineId[1]
	b[6] = machineId[2]
	// Pid, 2 bytes, specs don't specify endianness, but we use big endian. 进程id 占16位
	b[7] = byte(processId >> 8)
	b[8] = byte(processId)
	// Increment, 3 bytes, big endian  原子加1操作 占24位
	i := atomic.AddUint32(&objectIdCounter, 1) 
	b[9] = byte(i >> 16)
	b[10] = byte(i >> 8)
	b[11] = byte(i)
	return ObjectId(b[:])
}

```

matchID是主机名的md5值

```
// machineId stores machine id generated once and used in subsequent calls
// to NewObjectId function.
var machineId = readMachineId()

// readMachineId generates and returns a machine id.
// If this function fails to get the hostname it will cause a runtime error.
func readMachineId() []byte {
	var sum [3]byte
	id := sum[:]
	hostname, err1 := os.Hostname()//获取主机名
	if err1 != nil {
		n := uint32(time.Now().UnixNano())
		sum[0] = byte(n >> 0)
		sum[1] = byte(n >> 8)
		sum[2] = byte(n >> 16)
		return id
	}
	hw := md5.New()
	hw.Write([]byte(hostname))
	copy(id, hw.Sum(nil))//md5下
	return id
}
```

processId是进程号

```
var processId = os.Getpid() //获取进程号
```

对objectIdCounter进行原子+1操作，是一个累加值

```
// objectIdCounter is atomically incremented when generating a new ObjectId
// using NewObjectId() function. It's used as a counter part of an id.
var objectIdCounter uint32 = readRandomUint32()

// readRandomUint32 returns a random objectIdCounter.
func readRandomUint32() uint32 {
	// We've found systems hanging in this function due to lack of entropy.
	// The randomness of these bytes is just preventing nearby clashes, so
	// just look at the time.
	return uint32(time.Now().UnixNano())   // 由时间戳的毫秒来初始化
}

```