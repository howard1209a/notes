---
title: TCP黏包半包分析
date: 2023-10-29 12:41:51
tags: [计算机网络, 八股文]
---

# TCP 黏包半包分析

## 黏包/粘包

现象：发送 abc 和 def，接收 abcdef

原因：

- 操作系统层（TCP/IP 协议栈）
  - 发送端：一种情况是当程序不断地发送短字节，也就是向内核内存的发送缓冲区不断拷贝数据，由于通过网卡实际发送本身属于 IO 操作，因此速度肯定小于内存拷贝，因此短字节自然而然就在发送缓冲区中粘起来了。第二种情况是，linux 内核有一种 nagle 算法，算法为了节省报头损耗，会等待发送窗口填充满才会发送，如果发送的字节数短于发送窗口，就会等待接下来的数据“粘”过来。
  - 接收端：当第一段字节到达接收缓冲区的时候没有及时读出，等到第二段字节到达的时候，自然而然的就在接收缓冲区中粘起来了。
- 应用层：和各种语言的网络框架本身的应用层设计实现有关系，以 netty 为例，接收方 ByteBuf 设置太大就可能粘住。

## 半包/拆包

现象：发送 abcdef，接收 abc 和 def

原因：

- 操作系统层（TCP/IP 协议栈）
  - 发送端：一种情况是要发送的数据长度大于发送窗口长度，此时一定会拆包。第二种情况是，要发送的数据长度大于 MSS（MSS=mtu-40），此时数据一定会被拆分为多个 TCP 报文段，只要拆分成多个 TCP 报文段，就有可能导致最终结果拆包。
  - 接收端：如果一次只读出接收缓冲区中的一段数据，就有可能造成拆包。
- 应用层：和各种语言的网络框架本身的应用层设计有关系，以 netty 为例，接收方 ByteBuf 小于实际发送数据量就有可能拆包。

## 解决方案

- 短连接：每次要发送一条数据都经历：建立连接、发送数据、断开连接的过程。因为只发送一条数据，所以不会出现粘包，但可能出现拆包。
- 定长消息：服务端和客户端协定一个定长消息长度，客户端所有单个数据长度都小于等于这个最大长度，长度不够的补特定符号。这样发送端无论出现了各种粘和拆，但是接收端始终按定长收，就不会出现任何粘包和拆包。
- 分隔符：发送端在每条数据的前后设置分隔符，这样发送端无论出现了各种粘和拆，但是接收端始终依照分隔符接收，就不会出现任何粘包和拆包。分隔符的方式接收端要一个字节一个字节的查，效率非常低。
- 携带消息长度信息：每次发送消息时，先发送消息长度再发消息内容，这样客户端可以按照长度直接切出消息内容。是比较好的方式。
