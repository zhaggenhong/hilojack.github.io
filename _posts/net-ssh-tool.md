---
layout: page
title:	linux ssh
category: blog
description: 
---
# Preface
管理多台ssh 可以参考[pssh](/p/linux-pssh)

# Server
OpenSSH Installations under CentOS Linux

	# To install the server and client type:
	yum -y install openssh-server openssh-clients

Start the service:

	chkconfig sshd on
	service sshd start

Or:

	ssh
		[-D [bind_address:]port] [-e escape_char] [-F configfile]
		[-L [bind_address:]port:host:hostport]

Make sure port 22 is opened:

	netstat -tulpn | grep :22

# Firewall Settings

	vi /etc/sysconfig/iptables

Add the line

	-A RH-Firewall-1-INPUT -m state --state NEW -m tcp -p tcp --dport 22 -j ACCEPT

If you want to restict access to 192.168.1.0/24, edit it as follows:

	-A RH-Firewall-1-INPUT -s 192.168.1.0/24 -m state --state NEW -p tcp --dport 22 -j ACCEPT

If your site uses IPv6, and you are editing ip6tables, use the line:

	-A RH-Firewall-1-INPUT -m tcp -p tcp --dport 22 -j ACCEPT

Save and close the file. Restart iptables:

	service iptables restart

# OpenSSH Server Configuration

	vi /etc/ssh/sshd_config

	# To disable root logins, edit or add as follows:
	PermitRootLogin no

	#Restrict login to user tom and jerry only over ssh:
	AllowUsers tom jerry

	#Change ssh port i.e. run it on a non-standard port like 1235(default listen: 22)
	Port 1235

Save and close the file. Restart sshd:

	service sshd restart

# client

## config
以下 `~/.ssh/config` 包含了可以避免在特定网络环境中连接被断掉的情况的设置、使用压缩（这对于通过低带宽连接使用 scp 很有用），以及使用一个本地控制文件来开启到同一台服务器的多通道：

	TCPKeepAlive=yes
	ServerAliveInterval=15
	ServerAliveCountMax=6
	Compression=yes
	ControlMaster auto
	ControlPath /tmp/%r@%h:%p
	ControlPersist yes

## scp & sftp
openssh 包 包含这两个程序

### scp

如果我们想要 从远端系统，remote-sys 的家目录下复制文档 document.txt，到我们本地系统的当前工作目录下， 可以这样操作：

From remote to local

	$ scp remote-sys:document.txt .
	$ scp hilo@remote-sys:document.txt .

	# recursive download
	$scp -r user@your.server.example.com:/path/to/dir /home/user/Desktop/
	

从本地到远端：

	$ scp .ssh/id_rsa.pub username@hostname:/home/username/.ssh/authorized_keys

### sftp
就是 sftp 不需要远端系统中运行 FTP 服务器。它仅仅要求 SSH 服务器。 这意味着任何一台能用 SSH 客户端连接的远端机器，也可当作类似于 FTP 的服务器来使用。 这里是一个样本会话：

	$ sftp remote-sys
	> bye

> 也可在浏览器中使用sftp:// 访问

## connect

	ssh username@host -p port
	ssh -v verbose

### connect X server
假设我们正坐在一台装有 Linux 系统， 叫做 linuxbox 的机器之前，且系统中运行着 X 服务器，现在我们想要在名为 remote-sys 的远端系统中 运行 xload 程序，但是要在我们的本地系统中看到这个程序的图形化输出。我们可以这样做：

	[me@linuxbox ~]$ ssh -X remote-sys
	me@remote-sys's password:
	Last login: Mon Sep 08 13:23:11 2008
	[me@remote-sys ~]$ xload

这个 xload 命令在远端执行之后，它的窗口就会出现在本地。在某些系统中，你可能需要 使用 “－Y” 选项，而不是 “－X” 选项来完成这个操作。

## excute command

	ssh user@host bash < my.sh $1 $2 
	ssh user@host 'cat file.txt; ls'
	//host 后的所有参数会作为cmd 发给远程服务器，所以不用加引号
	ssh user@host cat file.txt; ls

	echo str | ssh user@server 'cat'

## non interactive login

### 利用RSA公私钥作验证
原理： 当ssh登录时,ssh程序会发送私钥去和服务器上的公钥做匹配.如果匹配成功就可以登录了
过程：
1.生成自己的公钥与私钥

	$ ssh-keygen -t rsa
	//or
	$ ssh-keygen -t rsa -P "passwd" -f ~/.ssh/id_rsa -C 'x@qq.com'
	
	# 改变权限，必须
	chmod 700 ~/.ssh
	cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
	# 改变权限，必须
	chmod 600 ~/.ssh/authorized_keys
	chmod 600 -R ~/.ssh/*

2.通过scp 或者ssh-copy-id 把公钥pub 传到远程

	$ scp .ssh/id_rsa.pub [-P 22] username@hostname:/home/username/.ssh/authorized_keys
	#或者用这个自动将公钥(.ssh/id_rsa.pub) 传到远程服务器(~/.ssh/authorized_keys)
	ssh-copy-id -i ~/.ssh/id_rsa.pub -P 22 username@hostname # 这个不是通用的命令，mac 下就没有
	#或者手动copy：
	cat ~/.ssh/id_rsa.pub | ssh user@machine "mkdir ~/.ssh; cat >> ~/.ssh/authorized_keys"

小结: 其实你可以一步到位：

	ssh-keygen -t rsa; ssh-copy-id user@host;

> authorized_keys 644，.ssh 700

#### 检验

	$ ssh -vT git@github.com-hilolt
	OpenSSH_6.2p2, OSSLShim 0.9.8r 8 Dec 2011
	debug1: Reading configuration data /Users/hilojack/.ssh/config
	debug1: /Users/hilojack/.ssh/config line 12: Applying options for github.com-hilolt
	debug1: Reading configuration data /etc/ssh_config
	debug1: identity file /Users/hilojack/.ssh/id_rsa_hilolt type 1

可参考：
http://www.cnblogs.com/weafer/archive/2011/06/10/2077852.html
http://www.chinaunix.net/old_jh/4/343905.html

### 利用sshpass

	sshpass -p "passwd" ssh yourUserName@host
	sshpass -p "passwd" ssh -l yourUserName host

### rsync
rsync 是一个同步命令，它是通过ssh 来下的，如果同步时不想输入密码:

	rsync --rsh='sshpass -p password ssh -l username' host.example.com:path

### sshpass
如果想scp 不输入密码, 可以借助sshpass:

cat ssh_login.sh:

	#!/usr/bin/env bash
	sshpass -p password ssh "$@"

Then:

	scp -S ssh_login.sh file user@host:path

### expect

	#!/usr/bin/expect
	spawn ssh -o StrictHostKeyChecking=no -l username hostname
	expect "*password:"
	send "password\r"
	interact

# Security
参考[ssh security]，保护你的ssh

1. 将port 从22 改成其它端口
2. 定义有限的用户列表, 利用PAM API(Pluggable Authentication Modules)
3. 要求提供特殊的"序列"识别用户: 端口敲门, 必须依次访问某些特定的端口序列，才能进入系统

# Reference
- [ssh copy]
- [ssh security]

[ssh copy]: http://ahei.info/ssh-copy-id.htm
[ssh security]:
http://www.ibm.com/developerworks/cn/aix/library/au-sshlocks/

