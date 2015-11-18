---
layout: page
title:	防火墙
category: blog
description:
---
# Preface
On [mac](/p/net-iptables-mac.md)

# 防火墙

	cat  /etc/sysconfig/iptables    (命令执行的规则只是临时生效)
	*filter
	:INPUT ACCEPT [0:0]
	:FORWARD ACCEPT [0:0]
	#允许所有本机向外的访问
	iptables -A OUTPUT -j ACCEPT
	# 允许所有对外访问简写（相当于-A OUTPUT -j ACCEPT)
	:OUTPUT ACCEPT [0:0]
	# 只允许对外端口1024:166083
	:OUTPUT ACCEPT [1024:166083]
	# 允许已经建立的或相关的通行
	-A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

	-A INPUT -p icmp -j ACCEPT
	-A INPUT -i lo -j ACCEPT
	# 允许80端口
	-A INPUT –m state –-state NEW –m tcp –p tcp –-dport 80 –j ACCEPT

## 保存规则

	# 命令行执行（临时）
	iptables -A INPUT –m state –-state NEW –m tcp –p tcp –-dport 80 –j ACCEPT
	service iptables save
	# 或者直接在配置中写规则
	cat /etc/sysconfig/iptables

	## 重启防火墙
	service iptables restart

	##
	service iptables stop
	/etc/init.d/iptables stop

## 规则查看
以下都可以

	iptables -L
	iptables --line -vnL
	/etc/init.d/iptables status

查看是否运行:

	service iptables status

## 汇总(archlinux)
通过命令行添加规则，配置文件不会自动改变，所以必须手动保存：

	# iptables-save > /etc/iptables/iptables.rules

修改配置文件后，需要重新加载服务：

	# systemctl reload iptables

或者通过 iptables 直接加载：

	# iptables-restore < /etc/iptables/iptables.rules

# 基本概念
> 参考：https://wiki.archlinux.org/index.php/Iptables_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#.E8.A7.84.E5.88.99_.EF.BC.88Rules.EF.BC.89

iptables 可以检测、修改、转发、重定向和丢弃 IPv4 数据包。

1. 过滤 IPv4 数据包的代码已经内置于内核中，并且按照不同的目的被组织成 `表` 的集合。
2. 表 由一组预先定义的 `链` 组成，链 包含遍历`顺序规则`。
3. 每一条规则包含一个谓词的潜在匹配和相应的`动作（称为 目标）`，如果谓词为真，该动作会被执行。也就是说条件匹配。

理解 iptables 如何工作的关键是这张图。
	![](/img/iptables_traverse.jpg)
图中在上面的小写字母代表 表，在下面的大写字母代表 链。从任何网络端口 进来的每一个 IP 数据包都要从上到下的穿过这张图。

在大多数使用情况下都不会用到 raw，mangle 和 security 表。下图简要描述了网络数据包通过 iptables 的过程：

								   XXXXXXXXXXXXXXXXXX
								 XXX     Network    XXX
								   XXXXXXXXXXXXXXXXXX
										   +
										   |
										   v
	 +-------------+              +------------------+
	 |table: filter| <---+        | table: nat       |
	 |chain: INPUT |     |        | chain: PREROUTING|
	 +-----+-------+     |        +--------+---------+
		   |             |                 |
		   v             |                 v
	 [local process]     |           ****************          +--------------+
		   |             +---------+ Routing decision +------> |table: filter |
		   v                         ****************          |chain: FORWARD|
	****************                                           +------+-------+
	Routing decision                                                  |
	****************                                                  |
		   |                                                          |
		   v                        ****************                  |
	+-------------+       +------>  Routing decision  <---------------+
	|table: nat   |       |         ****************
	|chain: OUTPUT|       |               +
	+-----+-------+       |               |
		  |               |               v
		  v               |      +-------------------+
	+--------------+      |      | table: nat        |
	|table: filter | +----+      | chain: POSTROUTING|
	|chain: OUTPUT |             +--------+----------+
	+--------------+                      |
										  v
								   XXXXXXXXXXXXXXXXXX
								 XXX    Network     XXX
								   XXXXXXXXXXXXXXXXXX

## todo
https://wiki.archlinux.org/index.php/Iptables_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#.E8.A7.84.E5.88.99_.EF.BC.88Rules.EF.BC.89
http://blog.chinaunix.net/uid-26212859-id-3483495.html

## 表（Tables）
iptables 包含 5 张表（tables）(`-t` 指定):

1. raw 用于配置数据包，raw 中的数据包不会被系统跟踪。
1. filter 是用于存放所有与防火墙相关操作的默认表。
1. nat 用于 网络地址转换（例如：端口转发）。
1. mangle 用于对特定数据包的修改（参考 损坏数据包）。
1. security 用于 强制访问控制 网络规则（例如： SELinux -- 详细信息参考 该文章）。

大部分情况仅需要使用 filter 和 nat。其他表用于更复杂的情况——包括多路由和路由判定——已经超出了本文介绍的范围。

## 链 （Chains）
表由链组成，链是一些按顺序排列的规则的列表。

1. 默认的 filter 表包含 INPUT， OUTPUT 和 FORWARD 3条内建的链，这3条链作用于数据包过滤过程中的不同时间点，如该流程图所示。
2. nat 表包含PREROUTING， POSTROUTING 和 OUTPUT 链。
3. 使用 man 8 iptables 查看其他表中内建链的描述。

默认情况下，任何链中都没有规则。可以向链中添加自己想用的规则。

1. 链的默认规则通常设置为 ACCEPT，如果想确保任何包都不能通过规则集，那么可以重置为 DROP。
2. 默认的规则总是在一条链的最后生效，所以在默认规则生效前数据包需要通过所有存在的规则。
3. 用户可以加入自己定义的链，从而使规则集更有效并且易于修改。如何使用自定义链请参考 Simple stateful firewall。

## 规则 （Rules）
数据包的过滤基于 规则。

1. 规则由一个`目标`（数据包包匹配所有条件后的动作）和很多匹配（导致该规则可以应用的数据包所满足的条件）指定。
目标使用 `-j 或者 --jump` 选项指定。
	目标可以是用户定义的链（例如，如果条件匹配，跳转到之后的用户定义的链，继续处理）、
		如果目标是用户定义的链，并且数据包成功穿过第二条链，目标将移动到原始链中的下一个规则。
	可以是一个内置的特定目标或者是一个目标扩展。
		目标扩展可以被终止（像内置目标一样）或者不终止（像用户定义链一样）。详细信息参阅 man 8 iptables-extensions。
	内置目标是 ACCEPT， DROP， QUEUE 和 RETURN，目标扩展是 REJECT and LOG。
		如果目标是内置目标，数据包的命运会立刻被决定并且在当前表的数据包的处理过程会停止。

2. 一个规则的典型`匹配事项`是数据包进入的端口（例如：eth0 或者 eth1）、数据包的类型（ICMP, TCP, 或者 UDP）和数据包的目的端口。

## 遍历链 （Traversing Chains）
流程图上显示了在任何接口上收到的网络数据包按顺序穿过表的交通控制链的过程。

1. 第一个路由策略包括决定数据包的目的地是本地主机（这种情况下，数据包穿过 INPUT 链）或是其他（数据包穿过 FORWARD 链）。
2. 随后的路由策略包括决定给传出包分配哪个端口。
3. 通过路径上的每一条链，链上的每一条规则按顺序评估，无论何时匹配了一条规则，相应的目标/跳转动作将会执行。

最常用的3个目标是 ACCEPT, DROP 和跳转到用户定义的链。内置链可以有默认的策略，但是用户定义的链没有。

关于匹配：

1. 如果链中经过的每一条规则都不能提供完整匹配，那么数据包像这张图片上一样回落到调用链:
	![](/img/iptables_subtraverse.png)
2. 如果任何时候一个 DROP 目标的规则实现完全匹配，那么丢弃数据包，不进行进一步处理。
3. 如果一个数据包在链中被 ACCEPT，那么它也将被所有超集链 ACCEPT，并且不再遍历其他超集链。 然而，要注意，数据包将以正常的方式继续遍历其他表中的其他链。

## 模块 （Modules）
有许多模块可以用来扩展 iptables，例如 connlimit, conntrack, limit 和 recent。这些模块增添了功能，可以进行更复杂的过滤。

# 日志
LOG 目标可以用来记录匹配某个规则的数据包。和 ACCEPT 或 DROP 规则不同，进入 LOG 目标之后数据包会继续沿着链向下走。

创建 logdrop 链：

	 iptables -N logdrop

定义规则：

	 iptables -A logdrop -m limit --limit 5/m --limit-burst 10 -j LOG
	 iptables -A logdrop -j DROP

现在任何时候想要丢弃数据包并且记录该事件，只要跳转到 logdrop 链，例如：

	# iptables -A INPUT -m conntrack --ctstate INVALID -j logdrop

# 规则
参考：http://os.51cto.com/art/201103/249074_all.htm

Example:
更改第二条规则，使其拒绝 17500 端口的任何数据包：

	# iptables -R INPUT 2 -p tcp --dport 17500 -j REJECT --reject-with icmp-port-unreachable

## Rule list

	-A POLICY 向规则链中添加一条规则，默认被添加到末尾
	-I POLICY 向规则链中添加一条规则，默认被添加到最前
		-A INPUT 输入规则
		-A OUPUT 输出规则
		-A FORWARD 转发规则
		-A POSTROUTING 参考: http://blog.chinaunix.net/uid-26212859-id-3483495.html
		-A PREROUTING
		-I INPUT 5 插入到第5条

	-D POLICY 从规则链中删除规则，可以指定序号或者匹配的规则来删除
		iptables -D INPUT 3（按号码匹配）
		删除 filter 表 INPUT 链中的第三条规则（不管它的内容是什么）
	-R POLICY 替换规则
		iptables -R INPUT 9 -j ACCEPT
		将原来编号为 9 的INPUT规则内容替换为“-j ACCEPT”

	-P POLICY，设置某个链的默认规则
		iptables -P INPUT DROP
		设置 filter 表 INPUT 链的默认规则是 DROP

	-F 清空规则
		iptables -F
		清空 filter 表中的所有规则

	-p protocol  比对通讯协议
		-p用来指定协议可以是tcp，udp，icmp等也可以是数字的协议号，
		-p !tcp 反向匹配

	--tcp-flags  比对 TCP
		iptables -p tcp --tcp-flags SYN,FIN,ACK SYN
		TCP状态旗号包括：SYN（同步）、ACK（应答）、FIN（结束）、RST（重设）、URG（紧急）、PSH（强迫推送）
		等均可使用于参数中，除此之外还可以使用关键词 ALL 和 NONE 进行比对

	--icmp-type
		iptables -A INPUT -p icmp --icmp-type 8
		用来比对 ICMP 的类型编号，可以使用代码或数字编号来进行比对。 案例ICMP类型是:8

	-m limit --limit
		用来比对某段时间内封包的平均流量，上面的例子是用来比对每秒平均流量是否超过一次 3 个封包。
		iptables -A INPUT -m limit --limit 3/sec
		一次同时涌入的包超过5个
		-m limit --limit-burst 5
	-m mac --mac-source
		# 匹配来源网络接口地址（不可用于OUTPUT, POSTROUTING, 因为封包要送到网 卡后，才能由网卡驱动程序通过ARP通讯查出目的地的mac地址, 在此之前iptables 无法以得到distination-mac-source）
		-m mac --mac-source 00:00:00:00:00:01
	 -m owner
		 #用来匹配来自本机的封包
		 -m owner --uid-owner 500
		 -m owner --gid-owner 500
		#特定进程
		 -m owner --pid-owner 100
		#匹配特定连接(session id)
		-m owner --sid-owner 100
	-m mark
		#用来匹配封包是否被表示某个号码，当封包被 匹配成功时，我们可以透过 MARK 处理动作，将该封包标示一个号码，号码最大不可以超过 4294967296。
		iptables -t mangle -A INPUT -m mark --mark 1
	-m state --state Status
		匹配连接状态：NEW,ESTABLISHED,RELATED,INVALID

	-i --in-interface 进入网络接口
		可以指定eth0,lo 等网络接口
	-o --out-interface 流出网络接口
		-o !eth0


	-T 指定要操作的表，默认是filter (cat /proc/net/ip_tables_names)
	-N 新建用户自定义的规则链
	-X 删除用户自定义的规则链
	-j 采取的动作，
		-j ACCEPT，REJECT, DROP，SNAT，DNAT，MASQUERADE
		REJECT 会拒绝目标分组的进入，并给企图连接服务的用户返回一个 connection refused 的错误消息。DROP 会放弃分组，而对 telnet 用户不发出任何警告.
		ACCEPT： 将封包放行，进行完此处理动作后，将不再匹配其它规则，直接跳往下一个规则链(natostrouting)。
		REJECT： 拦阻该封包，并传送封包通知对方，可以传送的封包有几个选择：ICMP port-unreachable、ICMP echo-reply 或是tcp-reset(这个封包会要求对方关闭 连接)，进行完此处理动作后，将不再匹配其它规则，直接中断过滤程序。 范例如下：
			iptables -A FORWARD -p TCP --dport 22 -j REJECT --reject-with tcp-reset
		DROP： 丢弃封包不予处理，进行完此处理动作后，将不再匹配其它规则，直接中断过滤程序。
		REDIRECT： 将封包重新导向到另一个端口(PNAT)，进行完此处理动作后，将会继续匹配其它规则。 这个功能可以用来实现透明代理或用来保护 web 服务器。例如：
			iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-ports 8080
		MASQUERADE： 改写封包来源 IP 为防火墙 NIC IP，可以指定 port 对应的范围，进行完此处理动作后，直接跳往下一个规则 链(manglepostrouting)。这个功能与SNAT 略有不同，当进行 IP 伪装时，不需指定要伪装成哪个 IP，IP会从网卡直接读取，当使用拨 号接连时，IP通常是由ISP公司的DHCP 服务器指派的，这个时候 MASQUERADE 特别有用。范例如下：
			iptables -t nat -A POSTROUTING -p TCP -j MASQUERADE --to-ports 1024-31000
		LOG： 将封包相关讯息纪录在 /var/log 中，详细位置请查阅 /etc/syslog.conf 配置文件，进行完此处理动作后，将会继续匹配其规则。例如：
		iptables -A INPUT -p tcp -j LOG --log-prefix "INPUT packets"
		SNAT： 改写封包来源 IP 为某特定 IP 或 IP 范围，可以指定 port 对应的范围，进行完此处理动作后，将直接跳往下一个规则(mangleostrouting)。范例如下：
			iptables -t nat -A POSTROUTING -p tcp -o eth0 -j SNAT --to-source?194.236.50.155-194.236.50.160:1024-32000
		DNAT： 改写封包目的地 IP 为某特定 IP 或 IP 范围，可以指定 port 对应的范围，进行完此处理动作后，将会直接跳往下一个规则链(filter:input 或 filter:forward)。范例如下：
			iptables -t nat -A PREROUTING -p tcp -d 15.45.23.67 --dport 80 -j DNAT --to-destination 192.168.1.1-192.168.1.10:80-100
		MIRROR： 镜射封包，也就是将来源 IP 与目的地 IP 对调后，将封包送回，进行完此处理动作后，将会中断过滤程序。
		QUEUE： 中断过滤程序，将封包放入队列，交给其它程序处理。通过自行开发的处理程序，可以进行其它应用，例如：计算连接费用等。
		RETURN： 结束在目前规则链中的过滤程序，返回主规则链继续过滤，如果把自定义规则链看成是一个子程序，那么这个动作，就相当于提前结束子程序并返回到主程序中。
		MARK： 将封包标上某个代号，以便提供作为后续过滤的条件判断依据，进行完此处理动作后，将会继续匹配其它规则。范例如下：
			iptables -t mangle -A PREROUTING -p tcp --dport 22 -j MARK --set-mark 2

	-s 指定源地址
	--src-range
	-d 指定目的地址
	--dst-range
		-s 10.209.0.213/16 C网
		-s 0/0 全部(可以不写)
		-src-range 192.168.0.1-192.168.0.200

	--port 端口
	--sport源端口
	--dport目的端口，端口必须和协议一起来配合使用
		--sport 1024:65535
		! --sport 30:1024 反向匹配
		# 允许访问22端口
		iptables -A INPUT -p tcp --dport 22 -j ACCEPT

> iptables 防范syn/cc/ack攻击
http://os.51cto.com/art/201103/249369.htm	http://www.oschina.net/question/12_3612

### -m match
matchname 跟数个选项对

	match = -m matchname [per-match-options]
	target = -j targetname [per-target-options]

比如:

	-A INPUT -s 192.168.1.0/24 -p tcp -m tcp --dport 80 -j ACCEPT

#### -m limit
使用 -m limit，可以使用 --limit 来设置平均速率或者使用 --limit-burst 来设置起始触发速率。在上述 logdrop 例子中：

	iptables -A logdrop -m limit --limit 5/m --limit-burst 10 -j LOG

Appends a rule which will log all packets that pass through it. The first 10 consecutive packets will be logged, and from then on only 5 packets per minute will be logged.
添加一条记录所有通过其的数据包的规则。开始的连续10个数据包将会被记录，之后每分钟只会记录5个数据包。

## ftp
ftp 默认有两个端口，先用21端口做认证，再用20端口做文件传输

	#允许FTP服务的21和20端口
	iptables -A INPUT -p tcp --dport 21 -j ACCEPT
	iptables -A INPUT -p tcp --dport 20 -j ACCEPT

	###Allow FTP connections on port 21 incoming and outgoing
	-A INPUT -p tcp -s 10.209.0.214/24 --sport 1024:65535 -d 0/0 --dport 21 -m state --state NEW,ESTABLISHED -j ACCEPT
	-A OUTPUT -p tcp -s 0/0 --sport 21 -d 0/0 --dport 1024:65535 -m state --state ESTABLISHED -j ACCEPT

	###Allow FTP port 20 for active connections incoming and outgoing
	-A INPUT -p tcp --sport 1:65535 --dport 1:65535 -m state --state ESTABLISHED,RELATED -j ACCEPT
	-A OUTPUT -p tcp --sport 1:65535 --dport 1:65535 -m state --state ESTABLISHED -j ACCEPT

	###Finally allow FTP passive inbound traffic
	-A OUTPUT -p tcp --sport 20 --dport 1:65535 -m state --state ESTABLISHED,RELATED -j ACCEPT
	-A INPUT -p tcp --sport 1:65535 --dport 20 -m state --state ESTABLISHED -j ACCEPT


## http

	-A INPUT -p tcp -m tcp --dport 80 -m state --state NEW,ESTABLISHED -j ACCEPT
