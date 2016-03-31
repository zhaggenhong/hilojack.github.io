---
layout: page
title:
category: blog
description:
---
# Preface

l2tp
http://www.live-in.org/archives/818.html
pptp
http://sr1.me/think-when-god-laugh/2015/10/07/set-up-pptpd-server-of-openvz-platform-and-scientifically-surf-the-internet-in-china.html

# l2tp
第二层隧道协议L2TP(Layer 2 Tunneling Protocol)是一种工业标准的Internet隧道协议，它使用UDP的1701端口进行通信。L2TP本身并没有任何加密，但是我们可以使用IPSec对L2TP包进行加密。L2TP VPN比PPTP VPN搭建复杂一些。

# yum安装openswan、ppp、xl2tpd方法：添加epel源
yum install openswan ppp xl2tpd
yum install make gcc gmp-devel bison flex lsof

#编辑配置文件/etc/ipsec.conf：

cp /etc/ipsec.conf /etc/ipsec.conf.bak
vim /etc/ipsec.conf

#查找protostack=auto，修改为：

protostack=netkey

#在最后加入：

conn L2TP-PSK-NAT
    rightsubnet=vhost:%priv
    also=L2TP-PSK-noNAT

conn L2TP-PSK-noNAT
    authby=secret
    pfs=no
    auto=add
    keyingtries=3
    rekey=no
    ikelifetime=8h
    keylife=1h
    type=transport
    left=YOUR.SERVER.IP.ADDRESS
    leftprotoport=17/1701
    right=%any
    rightprotoport=17/%any

# “YOUR.SERVER.IP.ADDRESS”换成VPS的外网IP。其中一些设置含义可以参考/etc/ipsec.d/examples/l2tp-psk.conf文件的内容。

4、设置共享密钥PSK
编辑配置文件/etc/ipsec.secrets：

	vim /etc/ipsec.secrets

输入：

	YOUR.SERVER.IP.ADDRESS %any: PSK "YourSharedSecret"

5、修改包转发设置
复制以下两段代码在终端里运行：

	for each in /proc/sys/net/ipv4/conf/*
	do
		echo 0 > $each/accept_redirects
		echo 0 > $each/send_redirects
	done

	echo 1 >/proc/sys/net/core/xfrm_larval_drop

修改内核设置，使其支持转发，编辑/etc/sysctl.conf文件：

	vim /etc/sysctl.conf
	将“net.ipv4.ip_forward”的值改为1。

使修改生效：

	sysctl -p

6、重启IPSec：

	/etc/init.d/ipsec restart

查看系统IPSec安装和启动的正确性：

	ipsec verify

没有报[FAILED]就可以了
