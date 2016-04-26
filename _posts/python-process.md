---
layout: page
title:
category: blog
description:
---
# Preface

# fork
Cover these functions:
- os.fork()
- os.getpid()

Example

	import os

	print('Process (%s) start...' % os.getpid())
	# Only works on Unix/Linux/Mac:
	pid = os.fork()
	if pid == 0:
		print('I am child process (%s) and my parent is %s.' % (os.getpid(), os.getppid()))
	else:
		print('I (%s) just created a child process (%s).' % (os.getpid(), pid))

# multiprocessing func(跨平台)
fork 与子程序func 是分离的, 父进程同步子进程比较麻烦
multiprocessing.Process(func) 则可以直接传func:

1. child = multiprocessing.Process(func)
2. child.start()
2. child.join() 等待子进程结束后再继续往下运行，通常用于进程间的同步
4. child.terminate() 或者直接结束

	from multiprocessing import Process
	import os

	# 子进程要执行的代码
	def run_proc(name):
		print('Run child process %s (%s)...' % (name, os.getpid()))

	if __name__=='__main__':
		print('Parent process %s.' % os.getpid())
		p = Process(target=run_proc, args=('test',))
		print('Child process will start.')
		p.start()
		p.join()
		print('Child process end.')

结果

	Parent process 928.
	Process will start.
	Run child process test (929)...
	Process end.

创建子进程时，只需要传入一个执行函数和函数的参数，创建一个Process实例，用start()方法启动，这样创建进程比fork()还要简单。
join()方法可以等待子进程结束后再继续往下运行，通常用于进程间的同步。

# pool
如果要启动大量的子进程，可以用进程池的方式批量创建子进程：
multiprocessing.Pool(4) 比fork 简单

1. p = multiprocessing.Poll(4)
2. for i in range(5): p.apply_async(func, args=(i,))
2. child.join() 等待子进程结束后再继续往下运行，通常用于进程间的同步

	from multiprocessing import Pool
	import os, time, random

	def long_time_task(name):
		print('Run task %s (%s)...' % (name, os.getpid()))
		start = time.time()
		time.sleep(random.random() * 3)
		end = time.time()
		print('Task %s runs %0.2f seconds.' % (name, (end - start)))

	if __name__=='__main__':
		print('Parent process %s.' % os.getpid())
		p = Pool(4)
		for i in range(5):
			p.apply_async(long_time_task, args=(i,)) # 加5个任务
		print('Waiting for all subprocesses done...')
		p.close()
		p.join()
		print('All subprocesses done.')

执行结果如下：

	Parent process 669.
	Waiting for all subprocesses done...
	Run task 0 (671)...
	Run task 1 (672)...
	Run task 2 (673)...
	Run task 3 (674)...
	Task 2 runs 0.14 seconds.
	Run task 4 (673)...
	Task 1 runs 0.27 seconds.
	Task 3 runs 0.86 seconds.
	Task 0 runs 1.41 seconds.
	Task 4 runs 1.91 seconds.
	All subprocesses done.

代码解读：

对Pool对象调用join()方法会等待所有子进程执行完毕，调用join()之前必须先调用close()，调用close()之后就不能继续添加新的Process了。

请注意输出的结果，task 0，1，2，3是立刻执行的，而task 4要等待前面某个task完成后才执行，这是因为Pool的默认大小在我的电脑上是4，因此，最多同时执行4个进程。这是Pool有意设计的限制，并不是操作系统的限制。如果改成：

	p = Pool(5)
	# 就可以同时跑5个进程。

由于Pool的默认大小是CPU的核数，如果你不幸拥有8核CPU，你要提交至少9个子进程才能看到上面的等待效果。

# shell 子进程
很多时候，子进程并不是自身，而是一个外部进程。我们创建了子进程后，还需要控制子进程的输入和输出。

subprocess模块可以让我们非常方便地启动一个子进程，然后控制其输入和输出。

## exec python

	cat a.py | python
	cat a.py | python - arg1

Note: raw_input 将不会正常工作，We better use :

	python <(cat p.py)
	python p.py

## exec shell

### subprocess.check_output

    import subprocess
    res = subprocess.check_output("ls -l", shell=True) # 以shell 角析
    print(res.decode()) # 默认返回值为 bytes 类型
        -rw-r--r-- ...

### subprocess.Popen
Popen 是最基础的类, 它是非阻塞的(除非执行`.stdout.read()`)

	subprocess.Popen('sleep 61') #wrong
	subprocess.Popen(['sleep', '61']).pid; #ok

	subprocess.Popen(['echo','12'], stdout=subprocess.PIPE, stderr=subprocess.PIPE, stdin=subprocess.PIPE)

read output

	proc = subprocess.Popen(['ls','-l'],stdout=subprocess.PIPE)
	line = proc.stdout.readline();
	while line;
		print "test:", line.rstrip()
		line = proc.stdout.readline();

#### shell=True 时:
1. 只取第一个参数，且把参数当成shell 语句而非命令
2. 可以执行multiple commands

	subprocess.Popen('sleep 5;echo abc',shell=True, stdout=subprocess.PIPE).stdout.read()

#### subprocess.Popen communicate
如果子进程还需要输入，则可以通过communicate()方法输入(其实是向stdin 写)：

	import subprocess

	print('$ nslookup')
	p = subprocess.Popen(['nslookup'], stdin=subprocess.PIPE, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
	output, err = p.communicate(b'set q=mx\npython.org\nexit\n')
	print(output.decode('utf-8'))
	print(output.decode())
	print('Exit code:', p.returncode)

上面的代码相当于在命令行执行命令nslookup，然后手动输入：

	set q=mx
	python.org
	exit

运行结果如下：

	$ nslookup
	Server:        192.168.19.4
	Address:    192.168.19.4#53

	Non-authoritative answer:
	python.org    mail exchanger = 50 mail.python.org.

	Authoritative answers can be found from:
	mail.python.org    internet address = 82.94.164.166
	mail.python.org    has AAAA address 2001:888:2000:d::a6

	Exit code: 0

### via call
类似run, 但

1. 只返回errno

	>>from subprocess import call
	>>errno=call("ls -l", shell=True)
	total 240
	-rw-r--r--   1 hilojack  staff   1.6K Dec 16 13:26 a.txt

### via run
它封装(package)的`subprocess.Popen`

	>>> import subprocess

doesn't capture output

	>>> subprocess.run(["ls", "-l", "/dev/null"])
	crw-rw-rw-  1 root  wheel    3,   2 Dec 19 11:13 /dev/null
	CompletedProcess(args=['ls', '-l', '/dev/null'], returncode=0)

capture output

	>>> subprocess.run(["ls", "-l", "/dev/null"], stdout=subprocess.PIPE)
	CompletedProcess(args=['ls', '-l', '/dev/null'], returncode=0, stdout=b'crw-rw-rw-  1 root  wheel    3,   2 Dec 19 11:13 /dev/null\n')
	>>> subprocess.run(["ls", "-l", "/dev/null"], stdout=subprocess.PIPE).stdout
	crw-rw-rw-  1 root  wheel    3,   2 Dec 19 11:13 /dev/null

check error

	>>> subprocess.run("exit 1", shell=True)
	CompletedProcess(args='exit 1', returncode=1)

	>>> subprocess.run("exit 1", shell=True, check=True)
	Traceback (most recent call last):
	  File "<stdin>", line 1, in <module>
	  File "/usr/local/Cellar/python3/3.5.0/Frameworks/Python.framework/Versions/3.5/lib/python3.5/subprocess.py", line 711, in run
		output=stdout, stderr=stderr)
	subprocess.CalledProcessError: Command 'exit 1' returned non-zero exit status 1


# 进程间通信
Process之间肯定是需要通信的，操作系统提供了很多机制来实现进程间的通信。Python的multiprocessing模块包装了底层的机制，提供了Queue、Pipes等多种方式来交换数据。

## queue
其实管道也是一种queue

	from multiprocessing import Process, Queue
	q = Queue()
	q.put(value)
	q.get(True)	 # 阻塞
	q.get(False) # 非阻塞
	time.sleep(random.random())

### queue example
我们以Queue为例，在父进程中创建两个子进程，一个往Queue里写数据，一个从Queue里读数据：

	from multiprocessing import Process, Queue
	import os, time, random

	# 写数据进程执行的代码:
	def write(q):
	    print('Process to write: %s' % os.getpid())
	    for value in ['A', 'B', 'C']:
	        print('Put %s to queue...' % value)
	        q.put(value)
	        time.sleep(random.random())

	# 读数据进程执行的代码:
	def read(q):
	    print('Process to read: %s' % os.getpid())
	    while True:
	        value = q.get(True)
	        print('Get %s from queue.' % value)

	if __name__=='__main__':
	    # 父进程创建Queue，并传给各个子进程：
	    q = Queue()
	    pw = Process(target=write, args=(q,))
	    pr = Process(target=read, args=(q,))
	    # 启动子进程pw，写入:
	    pw.start()
	    # 启动子进程pr，读取:
	    pr.start()
	    # 等待pw结束:
	    pw.join()
	    # pr进程里是死循环，无法等待其结束，只能强行终止:
	    pr.terminate()

运行结果如下：

	Process to write: 50563
	Put A to queue...
	Process to read: 50564
	Get A from queue.
	Put B to queue...
	Get B from queue.
	Put C to queue...
	Get C from queue.

在Unix/Linux下，multiprocessing模块封装了fork()调用，使我们不需要关注fork()的细节。由于Windows没有fork调用，因此，multiprocessing需要“模拟”出fork的效果，父进程所有Python对象都必须通过pickle序列化再传到子进程去，所有，如果multiprocessing在Windows下调用失败了，要先考虑是不是pickle失败了。

# distribute process, 分布式进程
Process可以分布到多台机器上，而Thread最多只能分布到同一台机器的多个CPU上。

Python的`multiprocessing模块`不但支持多进程，其中`managers子模块`还支持把多进程分布到多台机器上。 由于managers模块封装很好，不必了解网络通信的细节

	from multiprocessing.managers import BaseManager


举个例子：
我们先看服务进程，服务进程负责启动Queue，把Queue注册到网络上，然后往Queue里面写入任务：

	# task_master.py

	import random, time, queue
	from multiprocessing.managers import BaseManager

	# 发送任务的队列:
	task_queue = queue.Queue()
	# 接收结果的队列:
	result_queue = queue.Queue()

	# 从BaseManager继承的QueueManager:
	class QueueManager(BaseManager):
	    pass

	# 把两个Queue都注册到网络上, callable参数关联了Queue对象:
	QueueManager.register('get_task_queue', callable=lambda: task_queue)
	QueueManager.register('get_result_queue', callable=lambda: result_queue)
	# 绑定端口5000, 设置验证码'abc':
	manager = QueueManager(address=('', 5000), authkey=b'abc')
	# 启动Queue:
	manager.start()
	# 获得通过网络访问的Queue对象:
	task = manager.get_task_queue()
	result = manager.get_result_queue()
	# 放几个任务进去:
	for i in range(10):
	    n = random.randint(0, 10000)
	    print('Put task %d...' % n)
	    task.put(n)
	# 从result队列读取结果:
	print('Try get results...')
	for i in range(10):
	    r = result.get(timeout=10)
	    print('Result: %s' % r)
	# 关闭:
	manager.shutdown()
	print('master exit.')

请注意，当我们在一台机器上写多进程程序时，创建的Queue可以直接拿来用，但是，在分布式多进程环境下，添加任务到Queue不可以直接对原始的task_queue进行操作，那样就绕过了QueueManager的封装，必须通过manager.get_task_queue()获得的Queue接口添加。

然后，在另一台机器上启动任务进程（本机上启动也可以）：

	# task_worker.py

	import time, sys, queue
	from multiprocessing.managers import BaseManager

	# 创建类似的QueueManager:
	class QueueManager(BaseManager):
	    pass

	# 由于这个QueueManager只从网络上获取Queue，所以注册时只提供名字:
	QueueManager.register('get_task_queue')
	QueueManager.register('get_result_queue')

	# 连接到服务器，也就是运行task_master.py的机器:
	server_addr = '127.0.0.1'
	print('Connect to server %s...' % server_addr)
	# 端口和验证码注意保持与task_master.py设置的完全一致:
	m = QueueManager(address=(server_addr, 5000), authkey=b'abc')
	# 从网络连接:
	m.connect()
	# 获取Queue的对象:
	task = m.get_task_queue()
	result = m.get_result_queue()
	# 从task队列取任务,并把结果写入result队列:
	for i in range(10):
	    try:
	        n = task.get(timeout=1)
	        print('run task %d * %d...' % (n, n))
	        r = '%d * %d = %d' % (n, n, n*n)
	        time.sleep(1)
	        result.put(r)
	    except Queue.Empty:
	        print('task queue is empty.')
	# 处理结束:
	print('worker exit.')

任务进程要通过网络连接到服务进程，所以要指定服务进程的IP。

现在，可以试试分布式进程的工作效果了。先启动task_master.py服务进程：

	$ python3 task_master.py
	Put task 3411...
	Put task 1605...
	Put task 1398...
	Put task 4729...
	Put task 5300...
	Put task 7471...
	Put task 68...
	Put task 4219...
	Put task 339...
	Put task 7866...
	Try get results...

task_master.py进程发送完任务后，开始等待result队列的结果。现在启动task_worker.py进程：

	$ python3 task_worker.py
	Connect to server 127.0.0.1...
	run task 3411 * 3411...
	run task 1605 * 1605...
	run task 1398 * 1398...
	run task 4729 * 4729...
	run task 5300 * 5300...
	run task 7471 * 7471...
	run task 68 * 68...
	run task 4219 * 4219...
	run task 339 * 339...
	run task 7866 * 7866...
	worker exit.

task_worker.py进程结束，在task_master.py进程中会继续打印出结果：

	Result: 3411 * 3411 = 11634921
	Result: 1605 * 1605 = 2576025
	Result: 1398 * 1398 = 1954404
	Result: 4729 * 4729 = 22363441
	Result: 5300 * 5300 = 28090000
	Result: 7471 * 7471 = 55815841
	Result: 68 * 68 = 4624
	Result: 4219 * 4219 = 17799961
	Result: 339 * 339 = 114921
	Result: 7866 * 7866 = 61873956

这个简单的Master/Worker模型有什么用？其实这就是一个简单但真正的分布式计算，把代码稍加改造，启动多个worker，就可以把任务分布到几台甚至几十台机器上，比如把计算n*n的代码换成发送邮件，就实现了邮件队列的异步发送。

Queue对象存储在哪？注意到task_worker.py中根本没有创建Queue的代码，所以，Queue对象存储在task_master.py进程中
