---
layout: page
title:
category: blog
description:
---
# Preface
# Reference
http://havee.me/mac/2015-10/system-integrity-protection-on-el-capitan.html
http://www.jianshu.com/p/0572336a0771

这两天大家纷纷将 OS X 系统升级到了 El Capitan，然后发现，一些注入的工具无法使用了，某些系统目录无法使用了，第三方未签名的 kext 无法加载了，问题一堆堆的。这是因为，Mac OS X 在 10.11 中全面启用了 System Integrity Protection (SIP) —— 系统完整性保护技术。

SIP 技术主要是用来限制 root 用户的权限，以提升系统的健壮性。

	File System Protection
	Runtime Protection 
	Kernel Extensions 

具体哪些目录受到保护，可以查看文件

	/System/Library/Sandbox/rootless.conf

不被保护的列表存储在

	/System/Library/Sandbox/Compatibility.bundle/Contents/Resources/paths

Mac 提供了内置 csrutil 配置来进行一些 SIP 的配置。在默认情况下，SIP 是开启状态，你可以用一下指令查看

	$ csrutil status
	System Integrity Protection status: enabled

可配置项如下，字面意思：

	Apple Internal
	Kext Signing
	Filesystem Protections
	Debugging Restrictions
	DTrace Restrictions
	NVRAM Protections


上面已经说过，Mac 提供了内置的 csrutil 工具来让用户进行一些配置，不过，你需要重启进入到 Recovery mode (Cmd + R on boot) 下进行操作。

csrutil 的一些常用命令

	csrutil clear           # 清除 SIP 用户配置，即开启默认的 SIP
	csrutil enable          # 开启 SIP
	csrutil disable         # 禁用 SIP
	csrutil status          # 查看当前 SIP 配置

关于 csrutil enable 可用参数为

	csrutil enable --no-internal --without kext --without fs --without fs --without debug --without dtrace --without nvram

譬如说，如果你需要某系统目录的读写权限，譬如 homebrew 全新安装的时候，需要创建 /usr/local 目录，那么你需要

	1.进入RecoveryHD(Cmd+R)
	打开 Terminal.app，输入 csrutil enable --without fs
	重启至正常系统下，打开 Terminal.app，安装 homebrew
	再次重启至 Recovery mode
	打开 Terminal.app，输入 csrutil enable
	重启
	如果想安装第三方的 kext，那么建议装在目录 /Library/Extensions/ 下。
