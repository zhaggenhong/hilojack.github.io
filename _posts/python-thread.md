---
layout: page
title:
category: blog
description:
---
# Preface

# threading
Python的标准库提供了两个模块：`_thread`和`threading`，_thread是低级模块，threading是高级模块，对_thread进行了封装。绝大多数情况下，我们只需要使用threading这个高级模块。

	import time, threading
	t = threading.Thread(target=loop, name='LoopThread', args = (arg1, arg2, ..))
	t.start()
	t.join()
	threading.current_thread().name

Python的threading模块有个current_thread()函数，它永远返回当前线程的实例。

1. 主线程实例的名字叫`MainThread`，
1. 子线程的名字在创建时指定，我们用`LoopThread`命名子线程。
1. 名字仅仅在打印时用来显示，完全没有其他意义，如果不起名字Python就自动给线程命名为`Thread-1，Thread-2……`


## Example
启动一个线程就是把一个函数传入并创建Thread实例，然后调用start()开始执行：

	import time, threading

	def loop():
		print('thread %s is running...' % threading.current_thread().name)
		n = 0
		while n < 5:
			n = n + 1
			print('thread %s >>> %s' % (threading.current_thread().name, n))
			time.sleep(1)
		print('thread %s ended.' % threading.current_thread().name)

	print('thread %s is running...' % threading.current_thread().name)
	t = threading.Thread(target=loop, name='LoopThread')
	t.start()
	t.join()
	print('thread %s ended.' % threading.current_thread().name)

执行结果如下：

	thread MainThread is running...
	thread LoopThread is running...
	thread LoopThread >>> 1
	thread LoopThread >>> 2
	thread LoopThread >>> 3
	thread LoopThread >>> 4
	thread LoopThread >>> 5
	thread LoopThread ended.
	thread MainThread ended

# lock thread
如果线程要修改全局变量，为防collision 冲突，可以加lock
线程因为获得了锁，因此其他线程不能同时执行change_it()，只能等待，直到锁被释放后，其它线程获得该锁以后才能改。

	balance = 0
	//创建一把锁
	lock = threading.Lock()

	def run_thread(n):
		for i in range(100000):
			# 先要获取锁:
			lock.acquire()
			try:
				# 放心地改吧:
				balance +=n
				balance -=n
			finally:
				# 改完了一定要释放锁:
				lock.release()

# 多核CPU
打开Mac OS X的Activity Monitor，或者Windows的Task Manager，都可以监控某个进程的CPU使用率。
我们可以监控到一个死循环线程会100%占用一个CPU。

如果有两个死循环线程，在多核CPU中，可以监控到会占用200%的CPU，也就是占用两个CPU核心。
要想把N核CPU的核心全部跑满，就必须启动N个死循环线程。

试试用Python写个死循环：

	import threading, multiprocessing

	def loop():
		x = 0
		while True:
			x = x ^ 1

	for i in range(multiprocessing.cpu_count()):
		t = threading.Thread(target=loop)
		t.start()

启动与CPU核心数量相同的N个线程，在4核CPU上可以监控到CPU占用率仅有102%，也就是仅使用了一核。
但是用C、C++或Java来改写相同的死循环，直接可以把全部核心跑满，4核就跑到400%，8核就跑到800%，为什么Python不行呢？

因为:
1. Python的线程虽然是真正的线程，但解释器执行代码时，有一个GIL锁：`Global Interpreter Lock`，任何Python线程执行前，必须先获得GIL锁，
2. 然后，每执行100条字节码，解释器就自动释放GIL锁，让别的线程有机会执行。

这个GIL全局锁实际上把所有线程的执行代码都给上了锁，所以，多线程在Python中只能交替执行，即使100个线程跑在100核CPU上，也只能用到1个核。

## CPython
GIL是Python解释器设计的历史遗留问题，通常我们用的解释器是官方实现的CPython，要真正利用多核，除非重写一个不带GIL的解释器。

所以，在Python中 如果一定要通过多线程利用多核

1. 那只能通过C扩展来实现，不过这样就失去了Python简单易用的特点。
2. 不过，也不用过于担心，Python虽然不能利用多线程实现多核任务，但可以通过多进程实现多核任务。多个Python进程有各自独立的GIL锁，互不影响

# ThreadLocal
在线程中传递归局部变量，就是在函数调用的时候，传递起来很麻烦：

	def process_student(name):
		std = Student(name)
		# std是局部变量，但是每个函数都要用它，因此必须传进去：
		do_task_1(std)
		do_task_2(std)

	def do_task_1(std):
		do_subtask_1(std)
		do_subtask_2(std)

每个函数一层一层调用都这么传参数？或者用全局数组

	global_dict = {}

	def std_thread(name):
		std = Student(name)
		# 把std放到全局变量global_dict中：
		global_dict[threading.current_thread()] = std
		do_task_1()
		do_task_2()

	def do_task_1():
		# 不传入std，而是根据当前线程查找：
		std = global_dict[threading.current_thread()]

它最大的优点是消除了std对象在每层函数中的传递问题，但是，每个函数获取std的代码有点丑

有没有更简单的方式？
ThreadLocal应运而生，不用查找dict，ThreadLocal帮你自动做这件事：

	import threading

	# 创建全局ThreadLocal对象:
	local_school = threading.local()

	def process_student():
	    # 获取当前线程关联的student:
	    std = local_school.student
	    print('Hello, %s (in %s)' % (std, threading.current_thread().name))

	def process_thread(name):
	    # 绑定ThreadLocal的student:
	    local_school.student = name
	    process_student()

	t1 = threading.Thread(target= process_thread, args=('Alice',), name='Thread-A')
	t2 = threading.Thread(target= process_thread, args=('Bob',), name='Thread-B')
	t1.start()
	t2.start()
	t1.join()
	t2.join()

执行结果：

	Hello, Alice (in Thread-A)
	Hello, Bob (in Thread-B)

ThreadLocal最常用的地方就是为每个线程绑定一个数据库连接，HTTP请求，用户身份信息等，这样一个线程的所有调用到的处理函数都可以非常方便地访问这些资源。
