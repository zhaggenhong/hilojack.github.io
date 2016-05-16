---
layout: page
title:
category: blog
description:
---
# Preface


# netstat
> http://billie66.github.io/TLCL/book/zh/chap17.html

netstat的输出结果可以分为两个部分：

一个是Active Internet connections，称为有源TCP连接，其中"Recv-Q"和"Send-Q"指%0A的是接收队列和发送队列。这些数字一般都应该是0。如果不是则表示软件包正在队列中堆积。这种情况只能在非常少的情况见到。

另一个是Active UNIX domain sockets，称为有源Unix域套接口(和网络套接字一样，但是只能用于本机通信，性能可以提高一倍)。
Proto显示连接使用的协议,RefCnt表示连接到本套接口上的进程号,Types显示套接口的类型,State显示套接口当前的状态,Path表示连接到套接口的其它进程使用的路径名。

## format

	-n 以数学显示地址，而非地址符号(比如用127.0.0.1 代替localhost)
	-o, --timers
       Include information related to networking timers
	-v
	   Tell the user what is going on by being verbose.

### linux only

	-p 显示进程信息(PID/PNAME)
	-e 显示扩展信息，例如uid等(linux only)

## loop

	-c <seconds> 每隔一个固定时间，执行该netstat命令(linux only)

## filter

### listening
list `ESTABLISHED` and `CONNECTED` by default

	-a (all)
		Show both listening and non-listening sockets.

linux only:

	-l 仅列出有在 Listening (监听) 的服務状态(linux only)

mac only:

	 -l    Print full IPv6 address.
	 -W    In certain displays, avoid truncating addresses even if this causes some fields to overflow.

### protocol
linux only

	-t (tcp)仅显示tcp相关选项
	-u (udp)仅显示udp相关选项
	-x 仅显示UNIX相关选项

MAC only:

	-p protocol
	netstat -ap tcp | grep ridmi (8000端口)
	netstat -anp tcp | grep 8000
	lsof -i :8000

[维基TCP/UDP 端口](http://zh.wikipedia.org/zh-cn/TCP/UDP%E7%AB%AF%E5%8F%A3%E5%88%97%E8%A1%A8)

### protocol statistics

	-s 按各个协议进行统计

### route

	-r 显示路由信息，路由表
	-i 显示网络接口
	route (mac不支持用route 显示路由表)

mac only:

		-ib    show the number of bytes in and out(mac only)
		-iR     show link-layer reachability information for a given interface
		-id     show drop packets number

## port and process
find which process occupy specified port:
http://www.cyberciti.biz/faq/what-process-has-open-linux-port/

1. netstat - a command-line tool that displays network connections, routing tables, and a number of network interface statistics.

	$ sudo netstat -tulpn |grep 8888
	tcp        0      0 10.13.130.47:8000           0.0.0.0:*                   LISTEN      10557/php
	$ ls -l /proc/10557/exe
	lrwxrwxrwx 1 hilojack hilojack 0 Jun  8 17:09 /proc/10557/exe -> /usr/bin/php

2. fuser - a command line tool to identify processes using files or sockets.

	# sudo fuser port/protocol
	# sudo fuser 80/tcp
	80/tcp:             3813
	# ls -l /proc/3813/exe
	lrwxrwxrwx 1 vivek vivek 0 2010-10-29 11:00 /proc/3813/exe -> /usr/bin/transmission

Task: Find Out Current Working Directory Of a Process

	$ ls -l /proc/3813/cwd
	lrwxrwxrwx 1 vivek vivek 0 2010-10-29 12:04 /proc/3813/cwd -> /home/vivek
	$ pwdx 3813
	lrwxrwxrwx 1 vivek vivek 0 2010-10-29 12:04 /proc/3813/cwd -> /home/vivek
