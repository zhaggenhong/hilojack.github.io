---
layout: page
title:
category: blog
description:
---
# Preface

# processe status
linux上进程有5种状态:

1. 运行(正在运行或在运行队列中等待)
2. 中断(休眠中, 受阻, 在等待某个条件的形成或接受到信号)
3. 不可中断(收到信号不唤醒和不可运行, 进程必须等待直到有中断发生)
4. 僵死(进程已终止, 但进程描述符存在, 直到父进程调用wait4()系统调用后释放)
5. 停止(进程收到SIGSTOP, SIGSTP, SIGTIN, SIGTOU信号后停止运行运行)

ps工具标识进程的5种状态码:

1. R 运行 runnable (on run queue)
1. S 中断 sleeping
1. D 不可中断 uninterruptible sleep (usually IO)
1. Z 僵死 a defunct (”zombie”) process
1. T 停止 traced or stopped

# ps

	ps -options
	ps [axu] -option

-Options:

	-a,a
		other users' processes
	-x,x
		include processes which do not have a controlling terminal.
	-e,-A,-ax,ax
		other users' processes, including those without controlling terminals

## filter

	-U user,user,...
	-p pid,pid,...

## format

### info field

	-f
		UID   PID  PPID   C STIME   TTY           TIME CMD
	-u,u
		USER  PID  %CPU %MEM      VSZ    RSS   TT  STAT STARTED      TIME COMM
	-l
		UID   PID  PPID   F CPU PRI NI       SZ    RSS WCHAN     S             ADDR TTY           TIME CMD
	-w,w
		Use 132 columns to display information.
		If the -w option is specified more than once(-ww,ww), ps will use as many columns as necessary without regard for your window size

#### specify field

	ps -o rss,vsz
	ps -L "list all field

### command
print command name (not path)

	ps -c
	ps c

### info note

	USER: 行程拥有者
	PID: pid
	%CPU: 占用的 CPU 使用率
	TTY: 终端的次要装置号码 (minor device number of tty)
	STAT: 该行程的状态:
		R: running
		S: sleeping
		D: uninterruptible sleep (usually IO)
		Z: zombie
		T: stop
	START: 行程开始时间
	TIME: 执行的时间
	COMMAND:所执行的指令

	PR — 进程优先级
	NI — nice值。负值表示高优先级，正值表示低优先级

#### MEM
rss 是进程占用的物理内存，但是所有进程加起来的rss 会超过物理内存（因为进程间有很多内存是共享的, 比如共享库）

	RSS: 占用的记忆体大小
	VSZ: 占用的虚拟记忆体大小

	rss       RSS    resident set size, the non-swapped physical memory that a task has used (in kiloBytes). (alias rssize, rsz).
	vsz       VSZ    virtual memory size of the process in KiB (1024-byte units).(alias vsize)

可以用pmap 查看进程所用的非共享物理内存(writeable/private)

	➜  ~  pmap -d 15102
	15102:   redis-server
	Address           Kbytes Mode  Offset           Device    Mapping
	0000000000400000     424 r-x-- 0000000000000000 0fc:00001 redis-server
	00007fd1fc77f000    1576 r-x-- 0000000000000000 0fc:00001 libc-2.12.so
	00007fd1fd3d6000       4 r---- 000000000001f000 0fc:00001 ld-2.12.so
	mapped: 40048K    writeable/private: 29064K    shared: 0K

top 中的MEM：

	RES — 进程使用的、未被换出的物理内存大小，单位kb。RES=CODE+DATA
	VIRT — 进程使用的虚拟内存总量，单位kb。VIRT=SWAP+RES
	SHR — 共享内存大小，单位kb(物理内存)

### sort

	-r      Sort by current CPU usage

# 查看进程

	ps -ef | grep <pid>

# pstree

	$ pstree -p 3027
	-+= 00001 root /sbin/launchd
	\-+= 03001 root /usr/sbin/httpd -D FOREGROUND
	\--- 03027 _www /usr/sbin/httpd -D FOREGROUND

# proc
proc(`man 5 proc`) 可以获取更详细的进程信息

	/proc/<pid>/cwd
	/proc/<pid>/port

## process status
可以查看进程的内存使用情况:

- 包括虚拟内存大小（VmSize），物理内存大小（VmRSS），数据段大小（VmData），栈的大小 （VmStk），代码段的大小（VmExe），共享库的代码段大小（VmLib）等等。
- VmData，VmStk，VmExe和VmLib之和并不等于VmSize。这是因为共享库函数的数据段没有计算进去（VmData仅包含程序的数据段，不包括共享库函数的数据段， 也不包括通过mmap映射的区域。VmLib仅包括共享库的代码段，不包括共享库的数据 段）

	cat /proc/<pid>/status 

## smaps
通过`/proc/<pid>/smaps`可以查看进程整个虚拟地址空间的映射情况，它的输出从低地址到高地址按顺序输出每一个映射区域的相关信息，如下所示：

	$ cat /proc/<pid>/smaps
	注意：rwxp中，p表示私有映射（采用Copy-On-Write技术）。 Size字段就是该区域的大小。

