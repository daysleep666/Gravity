---
layout: post
title:  "golang实现ping过程"
date:   2018-11-16 20:25:0
categories: 基础知识
---

```

type ICMP struct {
	Type        uint8   
	Code        uint8
	Checksum    uint16
	Identifier  uint16
	SequenceNum uint16
}

func main() {
	conn, err := net.Dial("ip4:icmp", "66.42.110.182")
	if err != nil {
		fmt.Println(err.Error())
		return
	}
	defer conn.Close()

	var icmp ICMP
	icmp.Type = 8 // 8->echo message  0->reply message
	icmp.Code = 0
	icmp.Checksum = 0 // 校验和
	icmp.Identifier = 123 // 标识符
	icmp.SequenceNum = 0  // 序列号
	var buffer bytes.Buffer
	err = binary.Write(&buffer, binary.BigEndian, icmp)
	if err != nil {
		fmt.Println(err)
		return
	}
	icmp.Checksum = CheckSum(buffer.Bytes())    // 写入校验和
	buffer.Reset()
	err = binary.Write(&buffer, binary.BigEndian, icmp) // 将全部数据写入buffer中
	if err != nil {
		fmt.Println(err)
		return
	}
	if _, err = conn.Write(buffer.Bytes()); err != nil { // 发送
		fmt.Println(err)
		return
	}
	fmt.Printf("发送icmp成功 %v", icmp)
}

func CheckSum(data []byte) uint16 {
	fmt.Println("data=", data)
	var sum uint32
	i := 0
	for ; i < len(data); i += 2 {
		sum += uint32(data[i])<<8 + uint32(data[i+1]) // 两个字节，前一个字节右移8位加上后一个字节，就是组成了一个32位的字节。因为一个字节是8位，所以要先转成长为32位的int，同时也是为了方便计算。
		//它们的和必须是32位以上，因为sum是一个累加值，累加结果可能会超过16位，发生溢出。
	}
	if i != len(data) {
		sum += uint32(data[i]) << 8 //如果是奇数需要在最后补0，加0省略
	}
	sum += sum >> 16    // 将超过十六位的部分加到最低位上。
	return uint16(^sum) // 取反
}


```