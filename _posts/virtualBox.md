---
layout: page
title:
category: blog
description:
---

# 全屏

# 网络

## share network

### host only
要求将真实环境和虚拟环境隔离开，这时你就可采用host-only模式。在host-only模式中，所有的虚拟系统是可以相互通信的，但虚拟系统和真实的网络是被隔离开的。

　　提示:在host-only模式下，虚拟系统和宿主机器系统是可以相互通信的，相当于这两台机器通过双绞线互连。

　　在host-only模式下，虚拟系统的TCP/IP配置信息(如IP地址、网关地址、DNS服务器等)，都是由VMnet1(host-only)虚拟网络的DHCP服务器来动态分配的。

这应该是最为灵活的方式，有兴趣的话可以进行各种网络实验。和nat唯一的不同的是，此 种方式下，没有地址转换服务，因此，模认情况下，虚拟机只能到主机访问，这也是hostonly的名字的意义。

默认情况下，也会有一个dhcp服务加载到vmnet1上。这样连接到vmnet1上的虚拟机仍然可以设置成dhcp，方便系统的配置.

是不是这种方式就没有办法连接到外网呢，当然不是，事实上，这种方式更为灵活，你可以使用自己的方式，从而达到最理想的配置，例如：

a。使用自己dhcp服务：首先停掉vmware自带的dhcp服务，使dhcp服务更为统一。

b。使用自己的nat,方便加入防火墙。windows host可以做nat的方法很多，简单的如windows xp的internet共享，复杂的如windows server里的nat服务。

c. 使用自己的防火墙。因为你可以完全控制vmnet1,你可以加入（或试验）防火墙在vmnet1和外网的网卡间。

### NAT
让虚拟系统借助NAT(网络地址转换)功能，通过宿主机器所在的网络来访问公网。
NAT模式下的虚拟系统的TCP/IP配置信息是由VMnet8(NAT)虚拟网络的DHCP服务器提供的，无法进行手工修改，因此虚拟系统也 就无法和本局域网中的其他真实主机进行通讯。

这种方式也可以实现Host OS与Guest OS的双向访问。但网络内其他机器不能访问Guest OS，Guest OS可通过Host OS用NAT协议访问网络内其他机器。

只有一个外网地址，此种方式很合适


### bridged adapter
将虚拟网卡桥接到一个物理网卡上面，和linux下一个网卡 绑定两个不同地址类似，实际上是将网卡设置为混杂模式，从而达到侦听多个IP的能力

	Interface: en0 Wifi(ApiPort)
	Adapter Type: Intel PRO/1000 MT desktop(82540EM)
	Promisuous Mode: Allow All
	MAC Address: 0800273***

## renew ip
1. release ip

	sudo dhclient -r [interface]

2. obtain ip

	sudo dhclient [interface]

Refer to: [](/p/linux-net)

## query Guest Ip

	# Update arp table
	for i in {1..255}; do ping -c 1 192.168.0.$i & done
	for i in {1..255}; do ping -c 1 10.209.0.$i & done

	# Find vm name
	VBoxManage list runningvms

	# Find MAC: subsitute vmname with your vm's name
	VBoxManage showvminfo vmname

	# Find IP: substitute vname-mac-addr with your vm's mac address in ':' notation
	arp -a | grep vmname-mac-addr
