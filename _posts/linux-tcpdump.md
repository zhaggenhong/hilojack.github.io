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

	-i interface
		lo0 监听lo0
	-nn
		遇到协议号或端口号时，不要将这些号码转换成对应的协议名称或端口名称。比如，我们希望显示21，而非tcpdump自作聪明的将它显示成FTP。
	-X
		把协议头和包内容都以16 进制显示出来（tcpdump会以16进制和ASCII的形式显示），这在进行协议分析时是绝对的利器。
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

	# tcpdump -i eth0 -l |awk '{print $1}' ;#打印时间
	tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
	listening on eth0, link-type EN10MB (Ethernet), capture size 65535 bytes
	22:56:57.571680
	22:56:57.572103
	22:56:57.599515

> 如果不加-l选项，那么只有全缓冲区满，才会输出一次

# -t 输出时不打印时间
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

	# tcpdump -i eth0 -w flowdata

flowdata 是一个binary! 查看的话需要借助`-r`

	sudo tcpdump -r flowdata

# filter

## filter help
packet filter syntax

	man pcap-filter

## protocol

	tcpdump -i lo0 <protocol>
		tcp, udp, arp, ip, ether,ip6,rarp 等, 但tcpdump 不深入到应用层协议(eg. http)

	tcpdump -i lo0 'tcp'

## dst & src
可以借助`or` `and`

	# tcpdump -i eth0 'dst 8.8.8.8'
	# tcpdump -i eth0 -c 3 'dst port 53 or dst port 80'

tcpdump还支持如下的类型：

	host：指定主机名或IP地址，例如’host roclinux.cn’或’host 202.112.18.34′
	net ：指定网络段，例如’arp net 128.3’或’dst net 128.3′
	portrange：指定端口区域，例如’src or dst portrange 6000-6008′
		如果我们没有设置过滤类型，那么默认是host。

# Reference
- [tcpdump by roclinux]

[tcpdump by roclinux]: http://roclinux.cn/?p=2474
