---
layout: page
title:	tcpdump
category: blog
description:
---
# Preface
http://www.cnblogs.com/ggjucheng/archive/2012/01/14/2322659.html
学术的说，tcpdump是一种嗅探器（sniffer），利用以太网的特性，通过将网卡适配器（NIC）置于混杂模式（promiscuous）来获取传输在网络中的信息包。

tcpdump可以分为三大部分内容，第一是“选项”，第二是“过滤表达式”，第三是“输出信息”。

> 要用tcpdump抓包，一定要切换到root账户下，因为只有root才有权限将网卡变更为“混杂模式”。

# exmaple

	sudo tcpdump -i lo0 -nn -X  -c 1 port 8000
	sudo tcpdump -i en0 -nn -X  -c 1 port 8000

	-i interface
		lo0 监听lo0
	-nn
		遇到协议号或端口号时，不要将这些号码转换成对应的协议名称或端口名称。比如，我们希望显示21，而非tcpdump自作聪明的将它显示成FTP。
	-X
		把协议头和包内容都以16 进制显示出来（tcpdump会以16进制和ASCII的形式显示），这在进行协议分析时是绝对的利器。
	-A
		选项，则tcpdump只会显示ASCII形式的数据包内容，不会再以十六进制形式显示；

	-XX
		选项，则tcpdump会从以太网部分就开始显示网络包内容，而不是仅从网络层协议开始显示。一般不需要这么低层
	-c count
		只抓一个包
	port 8000
		监听8000 端口

结果示例：

	listening on lo0, link-type NULL (BSD loopback), capture size 65535 bytes
	19:31:52.674271 IP 127.0.0.1.60474 > 127.0.0.1.8000: Flags [S], seq 3581460068, win 65535, options [mss 16344,nop,wscale 5,nop,nop,TS val 652417598 ecr 0,sackOK,eol], length 0
		0x0000:  4500 0040 a548 4000 4006 0000 7f00 0001  E..@.H@.@.......
		0x0010:  7f00 0001 ec3a 1f40 d578 be64 0000 0000  .....:.@.x.d....
		0x0020:  b002 ffff fe34 0000 0204 3fd8 0103 0305  .....4....?.....
		0x0030:  0101 080a 26e3 1a3e 0000 0000 0402 0000  ....&..>........

	1 packet captured
	15 packets received by filter
	0 packets dropped by kernel

# 以太网信息

	-e 增加以太网信息
	sudo tcpdump -i lo0 -nn -c 1 -e

它会多一些这样的以太信息：

	AF IPv4 (2)
	OUI，即Organizationally unique identifier，是“组织唯一标识符”，在任何一块网卡（NIC）中烧录的6字节MAC地址中，前3个字节体现了OUI，其表明了NIC的制造组织。通常情况下，该标识符是唯一的。

回顾一下以太网塄我的头格式：

	struct sniff_ethernet {
		u_char ether_dhost[ETHER_ADDR_LEN]; /* 目的主机的地址 */
		u_char ether_shost[ETHER_ADDR_LEN]; /* 源主机的地址 */
		u_short ether_type; /* IP? ARP? RARP? etc */
	};

# -l 输出行变为行缓冲
-l选项的作用就是将tcpdump的输出变为“行缓冲”方式，这样可以确保tcpdump遇到的内容一旦是换行符即将缓冲的内容输出到标准输出，以便于利用管道或重定向方式来进行后续处理。

众所周知，Linux/UNIX的标准I/O提供了全缓冲、行缓冲和无缓冲三种缓冲方式。标准错误是不带缓冲的，终端设备常为行缓冲，而其他情况默认都是全缓冲的。

大家在使用tcpdump时，有时会有这样的需求：“对于tcpdump输出的内容，提取每一行的第一个域，即”时间域”，并输出出来，为后续统计所用”，这种场景下，我们就需要使用到-l来将默认的全缓冲变为行缓冲了。

	$ tcpdump -i eth0 -l |awk '{print $1}' ;#打印时间
	tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
	listening on eth0, link-type EN10MB (Ethernet), capture size 65535 bytes
	22:56:57.571680
	22:56:57.572103
	22:56:57.599515

> 如果不加-l选项，那么只有全缓冲区满，才会输出一次

# Info detail

## -t 输出时不打印时间
不打印开始的时间

	➜  test  sudo tcpdump -i lo0 -c 1
	20:42:28.372742 IP 10.209.12.156.61361 > 10.209.12.156.vcom-tunnel: Flags [S], seq 3578874752, win 65535, options [mss 16344,nop,wscale 5,nop,nop,TS val 656096720 ecr 0,sackOK,eol], length 0

	➜  test  sudo tcpdump -i lo0 -c 1 -t
	IP 10.209.12.156.61360 > 10.209.12.156.vcom-tunnel: Flags [S], seq 3383994986, win 65535, options [mss 16344,nop,wscale 5,nop,nop,TS val 656089608 ecr 0,sackOK,eol], length 0

## -v, 更详细的信息
加了-v选项之后，在原有输出的基础之上，你还会看到tos值、ttl值、ID值、总长度、校验值等。

	$ sudo tcpdump -i lo0 -c 1 -t -v
	IP (tos 0x0, ttl 64, id 30913, offset 0, flags [DF], proto TCP (6), length 64, bad cksum 0 (->931d)!)
    10.209.12.156.61400 > 10.209.12.156.vcom-tunnel: Flags [S], cksum 0x2f0c (incorrect -> 0x560f), seq 1836931889, win 65535, options [mss 16344,nop,wscale 5,nop,nop,TS val 656308489 ecr 0,sackOK,eol], length 0

`-vv` `-vvv`:

	-vv   Even  more  verbose output.  For example, additional fields are printed from
		  NFS reply packets, and SMB packets are fully decoded.

	-vvv  Even more verbose output.  For example, telnet SB ... SE options are printed
		  in full.  With -X Telnet options are printed in hex as well

## -F 指定过滤表达式所在的文件
如果过滤表达式有多个，可以将其写到一个文件`filter.txt`中，然后用`-F filter.txt` 指定

	$ cat filter.txt
	tcp
	port 8001

## -w -r 流量保存与回放
保存流量：

	$ tcpdump -i eth0 -w flowdata

flowdata 是一个binary! 查看的话需要借助`-r`

	sudo tcpdump -r flowdata

# tcp
分析tcp 包时，不要限制`dst or src`：

	sudo tcpdump -i en5 -nn -A host hilojack.com

## 三次握手

## 绝对序列号seq
tcp在收到第一条数据包之后，后续的数据包，是使用之前数据包的偏移来显示的，这个“1” 就是seq 4071349707的偏移1，也就是4071349708.
如果一开始不习惯这样，同时验证这个说法，我们可以使用tcpdump -i lo -S 来打印出绝对序列号


# filter

	'condition1 and condition2 or condtion3...'

## logic

	and not or

## filter help
packet filter syntax

	man pcap-filter

> 你会发现，过滤表达式大体可以分成三种过滤条件，“类型”、“方向”和“协议”，这三种条件的搭配组合就构成了我们的过滤表达式。

## protocol

	tcpdump -i lo0 <protocol>
		tcp, udp, arp, ip, ether,ip6,rarp 等, 但tcpdump 不深入到应用层协议(eg. http)

	tcpdump -i lo0 'tcp'

## dst & src
可以借助`or` `and`, 限制源与目的机ip

	$ tcpdump -i eth0 'dst 8.8.8.8'
	$ tcpdump -i eth0 -c 3 'dst port 53 or dst port 80'

tcpdump还支持如下的类型：

	host：指定主机名或IP地址，例如
		’host roclinux.cn’或’host 202.112.18.34′
		'host roclinux.cn and (baidu.com or qiyi.com)'
	net ：指定网络段，例如
		’arp net 128.3’或’dst net 128.3′
	portrange：指定端口区域，例如
		’src or dst portrange 6000-6008′
		如果我们没有设置过滤类型，那么默认是host。

## port
我想获取使用ftp端口和ftp数据端口的网络包

	tcpdump 'port ftp or ftp-data'

“这个ftp、ftp-data到底对应哪个端口？除了ftp/ftp-data，还有哪些服务名称我可以直接用呢？”

在Linux系统中，/etc/services这个文件里面，就存储着所有知名服务和传输层端口的对应关系。这个对应关系是由IANA组织（the Internet Assigned Numbers Authority，互联网数字分配机构）来全权负责的，你可以到这个链接http://www.iana.org/assignments/port-numbers通过Web方式查到

## example
【例子4】- 我想获取roclinux.cn和baidu.com之间建立TCP三次握手中第一个网络包，即带有SYN标记位的网络包，另外，目的主机不能是qiyi.com

	tcpdump 'tcp[tcpflags] & tcp-syn != 0 and not dst host qiyi.com'

这个语句看着比较复杂，其实如果要把这段解释清楚的确不容易，需要你具备计算机网络专业知识才行。这个我会安排一章来讲。

【例子5】- 打印IP包长超过576字节的网络包

	tcpdump 'ip[2:2] > 576'

【例子6】- 打印广播包或多播包，同时数据链路层不是通过以太网媒介进行的。

	tcpdump 'ether[0] & 1 = 0 and ip[16] >= 224'

# filter protocol
协议过滤必须单独提出来讲:

	proto [offset:size]


proto就是protocol的缩写，表示这里要指定的是某种协议的名称，比如ip、tcp、ether。其实proto这个位置，总共可以指定的协议类型有15个之多，包括：

	ether – 链路层协议
	fddi – 链路层协议
	tr – 链路层协议
	wlan – 链路层协议
	ppp – 链路层协议
	slip – 链路层协议
	link – 链路层协议
	ip
	arp
	rarp
	tcp
	udp
	icmp
	ip6
	radio

offset用来指定数据报偏移量，表示从某个协议的数据报的第多少位开始提取内容，默认的起始位置是0；而size表示从偏移量的位置开始提取多少个字节，可以设置为1、2、4。

如果只设置了offset，而没有设置size，则默认提取1个字节。比如`ip[2:2]`，就表示提取出第3、4个字节；而ip[0]则表示提取ip协议头的第一个字节。

在我们提取了特定内容之后，我们就需要设置我们的过滤条件了，我们可用的“比较操作符”包括：`>，<，>=，<=，=，!=`，总共有6个。

## offset
有一些常用的偏移量，可以通过一些名称来代替，比如icmptype表示ICMP协议的类型域、icmpcode表示ICMP的code域，tcpflags则表示TCP协议的标志字段域。

更进一步的，对于ICMP的类型域，可以用这些名称具体指代：

	icmp-echoreply, icmp-unreach, icmp-sourcequench, icmp-redirect, icmp-echo, icmp-routeradvert, icmp-routersolicit, icmp-timxceed, icmp-paramprob, icmp-tstamp, icmp-tstampreply, icmp-ireq, icmp-ireqreply, icmp-maskreq, icmp-maskreply。

而对于TCP协议的标志字段域，则可以细分为:

	tcp-fin, tcp-syn, tcp-rst, tcp-push, tcp-ack, tcp-urg。

# tip
之前没有提到的“小秘籍”，让大家在追查网络问题、进行协议分析时，可以用得上。



- 使用如下命令，则tcpdump会列出所有可以选择的抓包对象。

	$ tcpdump -D
	1.eth0
	2.any (Pseudo-device that captures on all interfaces)
	3.lo

- 如果想查看哪些ICMP包中“目标不可达、主机不可达”的包，请使用这样的过滤表达式：

	icmp[0:2]==0x0301

- 要提取TCP协议的SYN、ACK、FIN标识字段，语法是：

	tcp[tcpflags] & tcp-syn
	tcp[tcpflags] & tcp-ack
	tcp[tcpflags] & tcp-fin

- 要提取TCP协议里的SYN-ACK数据包，不但可以使用上面的方法，也可以直接使用最本质的方法：

	tcp[13]==18

- 如果要抓取一个区间内的端口，可以使用portrange语法：

	tcpdump -i eth0 -nn 'portrange 52-55' -c 1  -XX

# unpack packet info
http://roclinux.cn/?p=2820

# Reference
- [tcpdump by roclinux]

[tcpdump by roclinux]:
http://roclinux.cn/?p=2474
