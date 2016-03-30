---
layout: page
title:	linux 下的Fork 与 Exec
category: blog
description:
---
# Preface
多进程的的内容包括进程控制, 进程通信, 进程本身的结构.

# exec
虽然exec和source都是在父进程中直接执行，但exec这个与source有很大的区别，source是执行shell脚本，而且执行后会返回以前的shell。而exec的执行不会返回以前的shell了，而是直接把以前登陆shell作为一个程序看待，在其上经行复制。

exec命令示例

    exec ls
    在shell中执行ls，ls结束后不返回原来的shell中了
    exec <file
    将file中的内容作为exec的标准输入
    exec >file
    将file作为标准写出
    exec 3<file
    将file读入到fd3中
    sort <&3
    fd3中读入的内容被分类
    exec 4>file
    将写入fd4中的内容写入file中
    ls >&4
    Ls将不会有显示，直接写入fd4中了，即上面的file中
    exec 5<&4
    创建fd4的拷贝fd5
    exec 3<&-
    关闭fd3

## exec的重定向

先上我们进如/dev/fd/目录下看一下：

    root@localhost:~/test#cd /dev/fd
    root@localhost:/dev/fd#ls
    0  1  2  255
    默认会有这四个项：
		0是标准输入，默认是键盘。
		1是标准输出，默认是屏幕/dev/tty
		2是标准错误，默认也是屏幕

当我们执行exec 3>test时：

    root@localhost:/dev/fd#exec 3>/root/test/test
    root@localhost:/dev/fd#ls
    0  1  2  255  3
    root@localhost:/dev/fd#

看到了吧，多了个3，也就是又增加了一个设备，这里也可以体会下linux设备即文件的理念。这时候fd3就相当于一个管道了，重定向到fd3中的文件会被写在test中。关闭这个重定向可以用exec 3>&-。

## 应用举例：

    exec 3<test
    while read -u 3 pkg
    do
    echo "$pkg"
    done

# kill 信号
kill -15 默认向进程发送GIGTERM 信号

	kill -l 用于显示所有信号
	kill -l SEGV 查看信号值

	HUP    1    终端断线(如果忽略HUP, 终端关闭时进程不会退出)
	INT     2    中断（同 Ctrl + C）
	QUIT    3    退出（同 Ctrl + \）
	TERM   15    终止(默认)
	KILL    9    强制终止
	CONT   18    继续（与STOP相反， fg/bg命令）
	STOP    19    暂停（同 Ctrl + Z）
	SEGV	11	segmentfault

# Rerference
- [memory]
- [fork]

[fork]: http://www.cnblogs.com/hicjiajia/archive/2011/01/20/1940154.html "x1"
[memory]: http://arhythmically40.rssing.com/chan-19541204/all_p4.html#item76
[fork exec]: http://www.cnblogs.com/hicjiajia/archive/2011/01/20/1940154.html
