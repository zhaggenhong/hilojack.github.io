---
layout: page
title:	
category: blog
description: 
---
# Preface

	nohup命令说明：
	用途：不挂断地运行命令。
	语法：nohup Command [ Arg ... ] [　& ]
	描述：nohup 命令运行由 Command 参数和任何相关的 Arg 参数指定的命令，忽略所有挂断（SIGHUP）信号。在注销后使用 nohup 命令运行后台中的程序。要运行后台中的 nohup 命令，添加 & （ 表示“and”的符号）到命令的尾部。

此时默认地程序运行的输出信息放到当前文件夹的nohup.out 文件中去，加不加&并不会影响这个命令。只是让程序前台或者后台运行而已。

http://www.wangyuxiong.com/archives/52065
