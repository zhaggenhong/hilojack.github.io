---
layout: page
title:	Linux Network
category: blog
description:
---
# Preface

查看网络的命令行工具非常多，而mac 下有一款功能完备网络查看工具：Network Utility

![](/img/linux-net-mac-network-utility.png)

# tool
网络相关的工具有:

	dns: ip 或 ifconfig，dig
	monitor: netstat -lntp 或 ss -plat
	socket: lsof 查看套接字和文件

	ss：套接口统计数据
	nc：网络调试和数据传输
	socat：套接字中继和 tcp 端口转发（类似 netcat）

# todo
网络监控工具:
http://mp.weixin.qq.com/s?__biz=MzA3MzYwNjQ3NA==&mid=205572993&idx=1&sn=9c05c68f6209ff990dd0da8e8a77e334&scene=1&from=groupmessage&isappinstalled=0#rd

# centos网络配置(手动设置，自动获取)的2种方法
作者:海底苍鹰
地址:http://blog.51yip.com/linux/1120.html

## 设置网络链接ifcfg-eth0

	cat /etc/sysconfig/network-scripts/ifcfg-eth0

内容如下：

	DEVICE=eth0                               //由eth0来启动
	BOOTPROTO=dhcp                     //获取IP的方式是自动获取，static是固定IP，none是手动
	#BOOTPROTO=none                         //启动为手动
	HWADDR=00:16:D3:A2:F6:09       //网卡的物理地址
	IPV6INIT=yes                              //是否支持IP6
	IPV6_AUTOCONF=yes                //IP6是否自动配置
	ONBOOT=yes                       //启动时网络接口是否有效

	#IPADDR=192.168.1.108                   //设置的IP
	#NETMASK=255.255.255.0                //子网掩码
	#TYPE=Ethernet                                //网络类型
	#GATEWAY=192.168.1.1                            //加上网关
	#DNS1=223.5.5.5	//加上主DNS
	#DNS2=223.6.6.6	//加上次DNS

[For more detail]( https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Deployment_Guide/s1-networkscripts-interfaces.html)fdsfdsfsb 	sf	sdfbsdf	a

## 初始化网络

	$ cat /etc/sysconfig/network
	NETWORKING=yes                                 //网络是否可用
	NETWORKING_IPV6=yes
	HOSTNAME=localhost.localdomain //主机名，主机名在/etc/hosts里面配置


	$ cat /etc/hosts
	127.0.0.1               localhost.localdomain localhost     //根/etc/resolv.conf中search要对应，localhost.localdomain
	::1             localhost6.localdomain6 localhost6

	$ cat /etc/resolv.conf
	#nameserver 223.5.5.5	//加上主DNS
	#nameserver 223.6.6.6	//加上次DNS
	#search localdomain //搜索要找的域名，在/etc/hosts里面设定的有(用于子域名)

## start & restart

	# ifdown eth0
	# ifup eth0
	### RHEL/CentOS/Fedora specific command ###
	# /etc/init.d/network restart

or via `dhclient`:

	# 1. release ip
	sudo dhclient -r [interface]
	# 2. obtain ip
	sudo dhclient [interface]
		-r
			release ip
		interface
			eth0 | wlan1 | ...
		-v
			verbose

Refer to : http://www.cyberciti.biz/faq/howto-linux-renew-dhcp-client-ip-address/

# netstat
/p/linux-net-netstat

# lsof
[/p/linux-process-lsof](/p/linux-process-lsof)

# ifconfig
ifconfig 是用于查看网卡接口的。
对于linux 来说，它会显示网关gateway(route), 而对于mac 它不会显示网关.
不过mac可以使用通过路由表netstat -nr | grep '^default' 查看到网关。或者通过mac 的特有命令`route get default` 得到mac 的网关信息

# route

 route -- manually manipulate the routing tables

## check route
check route tables

	$ netstat -nr
	$ route (for linux only)

check gateway(route)

	$ netstat -nr | grep default |grep -o -E '\d{1,3}(\.\d{1,3}){3}'
	$ route get default | grep gateway (for mac only)
	$ route get baidu.com | grep gateway (for mac only)
	$ ifconfig |grep gateway (for linux only)

## add route

	route add 1.0.1.0/24 "$gateway"
	route del 1.0.1.0/24 "$gateway"

## traceroute

	traceroute - print the route packets trace to network host

这个 traceroute 程序（一些系统使用相似的 tracepath 程序来代替）会显示从本地到目的主机 要经过的所有“跳数”的网络流量列表(路由)。

	$ traceroute m.weibo.cn
	traceroute: Warning: m.weibo.cn has multiple addresses; using 202.108.7.133
	traceroute to weibo.cn (202.108.7.133), 64 hops max, 52 byte packets
	 1  localhost (192.168.1.1)  2.170 ms  2.538 ms  1.578 ms
	 2  114.249.224.1 (114.249.224.1)  50.709 ms  134.246 ms  79.752 ms
	 3  61.51.246.165 (61.51.246.165)  5.262 ms  4.654 ms  4.134 ms
	 4  124.65.57.245 (124.65.57.245)  2.980 ms  3.936 ms  3.969 ms
	 5  61.148.143.22 (61.148.143.22)  2.058 ms  2.049 ms  3.188 ms
	 6  210.74.178.198 (210.74.178.198)  2.991 ms  4.344 ms  3.471 ms
	 7  202.108.7.133 (202.108.7.133)  1.834 ms  2.076 ms  2.080 ms

## mtr
mtr - a network diagnostic tool
mtr命令比traceroute要好, 提供更多信息

	mtr weibo.com

# DNS
DNS Utility:
1. host <domain> - DNS lookup Utility
1. dig [@dns_server] <domain> - DNS lookup Utility
1. nslookup hilojack.com

	host baidu.com
	dig baidu.com
	dig @223.5.5.5 baidu.com

Example:

	$ nslookup weibo.cn
	Server:		8.8.8.8
	Address:	8.8.8.8#53

	Non-authoritative answer:
	hilojack.com	canonical name = hilojack.github.io.
	hilojack.github.io	canonical name = github.map.fastly.net.
	Name:	github.map.fastly.net
	Address: 103.245.222.133

## list current dns
On Mac OSX:

	networksetup -getdnsservers Wi-Fi
	networksetup -getsearchdomains Wi-Fi

## set dns server
On Mac OSX

	networksetup -listallnetworkservices
	sudo networksetup -setdnsservers Wi-Fi 223.5.5.5 223.6.6.6

On Linux

	vi /etc/resolv.conf
	nameserver 8.8.8.8

## get ip
DNS查询，寻找域名domain对应的IP:

	$host domain

## get own ip
find ip

	ip=$(/sbin/ip -o -4 addr list eth0 | awk '{print $4}' | cut -d/ -f1)

# selinux
Refer to [linux-selinux](/p/linux-selinux)

# iptables, 防火墙
Refer to [linux-iptables](/p/linux-iptables)

# vpn route
打开 http://ip.chinaz.com 显示的是国内 IP，说明智能加速安装成功

## install

	sudo cp ip-up ip-down /etc/ppp/
	cd /etc/ppp; sudo chmod a+x ip-up ip-down

## uninstall

	sudo rm /etc/ppp/{ip-up,ip-down}

cat ip-up

	# Generate on 2014-07-02 04:38 by VPNCloud
	export PATH="/bin:/sbin:/usr/sbin:/usr/bin"
	OLDGW=`netstat -nr | grep '^default' | grep -v 'ppp' | sed 's/default *\([0-9\.]*\) .*/\1/'`
	if [ ! -e /tmp/pptp_oldgw ]; then
		echo "${OLDGW}" > /tmp/pptp_oldgw
	fi
	killall -HUP mDNSResponder # 或者 dscacheutil -flushcache
	route add 1.0.1.0/24 "${OLDGW}"
	route add 1.0.2.0/23 "${OLDGW}"
	route add 1.0.8.0/21 "${OLDGW}"
	route add 1.0.32.0/19 "${OLDGW}"
	route add 1.1.0.0/24 "${OLDGW}"
	route add 1.1.2.0/23 "${OLDGW}"
	route add 1.1.4.0/22 "${OLDGW}"

http://igfvv.com/local-routing-table-for-mac-os-x
https://www.ytvpn.com/admin/speed_up


内网
ip-down 里增加：

	route delete 172.16.0.0/12 ${OLDGW}
	route delete 10.0.0.0/8 ${OLDGW}
	route delete 192.168.0.0/16 ${OLDGW}
	route delete 202.108.0.0/16 ${OLDGW}

ip-up 里增加：

	route add 172.16.0.0/12 "${OLDGW}"
	route add 10.0.0.0/8 "${OLDGW}"
	route add 192.168.0.0/16 "${OLDGW}"
	route add 202.108.0.0/16 ${OLDGW}

# nmap
常见的网络扫描器有 SSS，X-Scan，Superscan等，功能最强大的当然是[Nmap](http://www.aiezu.com/system/linux/linux_nmap_tutorial.html)

Nmap命令的格式为：

	Nmap [ 扫描类型 ... ] [ 通用选项 ] { 扫描目标说明 }

下面对Nmap命令的参数按分类进行说明：

1. 扫描类型

	-sT	TCP connect()扫描，这是最基本的TCP扫描方式。这种扫描很容易被检测到，在目标主机的日志中会记录大批的连接请求以及错误信息。
	-sS	TCP同步扫描(TCP SYN)，因为不必全部打开一个TCP连接，所以这项技术通常称为半开扫描(half-open)。这项技术最大的好处是，很少有系统能够把这记入系统日志。不过，你需要root权限来定制SYN数据包。
	-sF,-sX,-sN	秘密FIN数据包扫描、圣诞树(Xmas Tree)、空(Null)扫描模式。这些扫描方式的理论依据是：关闭的端口需要对你的探测包回应RST包，而打开的端口必需忽略有问题的包(参考RFC 793第64页)。
	-sP	ping扫描，用ping方式检查网络上哪些主机正在运行。当主机阻塞ICMP echo请求包是ping扫描是无效的。nmap在任何情况下都会进行ping扫描，只有目标主机处于运行状态，才会进行后续的扫描。
	-sU	如果你想知道在某台主机上提供哪些UDP(用户数据报协议,RFC768)服务，可以使用此选项。
	-sA	ACK扫描，这项高级的扫描方法通常可以用来穿过防火墙。
	-sW	滑动窗口扫描，非常类似于ACK的扫描。
	-sR	RPC扫描，和其它不同的端口扫描方法结合使用。
	-b	FTP反弹攻击(bounce attack)，连接到防火墙后面的一台FTP服务器做代理，接着进行端口扫描。

2. 通用选项

	-P0	在扫描之前，不ping主机。
	-PT	扫描之前，使用TCP ping确定哪些主机正在运行。
	-PS	对于root用户，这个选项让nmap使用SYN包而不是ACK包来对目标主机进行扫描。
	-PI	设置这个选项，让nmap使用真正的ping(ICMP echo请求)来扫描目标主机是否正在运行。
	-PB	这是默认的ping扫描选项。它使用ACK(-PT)和ICMP(-PI)两种扫描类型并行扫描。如果防火墙能够过滤其中一种包，使用这种方法，你就能够穿过防火墙。
	-O	这个选项激活对TCP/IP指纹特征(fingerprinting)的扫描，获得远程主机的标志，也就是操作系统类型。
	-I	打开nmap的反向标志扫描功能。
	-f	使用碎片IP数据包发送SYN、FIN、XMAS、NULL。包增加包过滤、入侵检测系统的难度，使其无法知道你的企图。
	-v	冗余模式。强烈推荐使用这个选项，它会给出扫描过程中的详细信息。
	-S <IP>	在一些情况下，nmap可能无法确定你的源地址(nmap会告诉你)。在这种情况使用这个选项给出你的IP地址。
	-g port	设置扫描的源端口。一些天真的防火墙和包过滤器的规则集允许源端口为DNS(53)或者FTP-DATA(20)的包通过和实现连接。显然，如果攻击者把源端口修改为20或者53，就可以摧毁防火墙的防护。
	-oN	把扫描结果重定向到一个可读的文件logfilename中。
	-oS	扫描结果输出到标准输出。
	--host_timeout	设置扫描一台主机的时间，以毫秒为单位。默认的情况下，没有超时限制。
	--max_rtt_timeout	设置对每次探测的等待时间，以毫秒为单位。如果超过这个时间限制就重传或者超时。默认值是大约9000毫秒。
	--min_rtt_timeout	设置nmap对每次探测至少等待你指定的时间，以毫秒为单位。
	-M count	置进行TCP connect()扫描时，最多使用多少个套接字进行并行的扫描。

3. 扫描目标

	目标地址	可以为IP地址，CIRD地址等。如192.168.1.2，222.247.54.5/24
	-iL filename	从filename文件中读取扫描的目标。
	-iR	让nmap自己随机挑选主机进行扫描。
	-p	端口 这个选项让你选择要进行扫描的端口号的范围。如：-p 20-30,139,60000。
	-exclude	排除指定主机。
	-excludefile	排除指定文件中的主机。

# ngrok 内网穿透
https://www.imququ.com/post/self-hosted-ngrokd.html?utm_source=tuicool

# Todo
计算机网络:自顶向下方法 http://www.amazon.cn/gp/product/B001TCBSJ0/ref=as_li_ss_tl?ie=UTF8&camp=536&creative=3132&creativeASIN=B001TCBSJ0&linkCode=as2&tag=jysperm07-23

# Reference
[could_not_bind_to_port]

[could_not_bind_to_port]: http://wiki.apache.org/httpd/CouldNotBindToAddress
