---
layout: page
title:
category: blog
description:
---
# Preface

# 非阻塞IO
在IO操作的过程中，当前线程被挂起，而其他需要CPU执行的代码就无法被当前线程执行了。

1. 多线程和多进程的模型虽然解决了并发问题，但切换线程的开销也很大，一旦线程数量过多，CPU的时间就花在线程切换上了
2. 另一种解决IO问题的方法是异步IO。它只发出IO指令，并不等待IO结果，然后就去执行其他代码了。一段时间后，当IO返回结果时，再通知CPU进行处理。

异步IO模型需要一个消息循环，在消息循环中，主线程不断地重复“读取消息-处理消息”这一过程：

	loop = get_event_loop()
	while True:
		event = loop.get_event()
		process_event(event)

消息模型其实早在应用在桌面应用程序中了。一个GUI程序的主线程就负责不停地读取消息并处理消息。所有的键盘、鼠标等消息都被发送到GUI程序的消息队列中，然后由GUI程序的主线程处理。

消息模型是如何解决同步IO必须等待IO操作这一问题的呢？当遇到IO操作时，代码只负责发出IO请求，不等待IO结果，然后直接结束本轮消息处理，进入下一轮消息处理过程。当IO操作完成后，将收到一条“IO完成”的消息，处理该消息时就可以直接获取IO操作结果。

# 协程
假设由协程执行，在执行A的过程中，可以随时中断，去执行B，B也可能在执行过程中中断再去执行A.
看起来A、B的执行有点像多线程，但协程的特点在于是一个线程执行，那和多线程比，协程有何优势？

1. 最大的优势就是协程极高的执行效率。因为子程序切换不是线程切换，而是由程序自身控制，因此，线程数量越多，协程的性能优势就越明显。
2. 第二大优势就是不需要多线程的锁机制，因为只有一个线程，也不存在同时写变量冲突，在协程中控制共享资源不加锁，只需要判断状态就好了，所以执行效率比多线程高很多。

因为协程是一个线程执行，那怎么利用多核CPU呢？最简单的方法是多进程+协程，既充分利用多核，又充分发挥协程的高效率，可获得极高的性能。

Python对协程的支持是通过generator实现的。

1. 在generator中，我们不但可以通过for循环来迭代，还可以不断调用next()函数获取由yield语句返回的下一个值。
2. 但是Python的yield不但可以返回一个值，它还可以接收调用者发出的参数。

## 语法

	r = c.send(n)
	c.close()

## 例子：
传统的生产者-消费者模型是一个线程写消息，一个线程取消息，通过锁机制控制队列和等待，但一不小心就可能死锁。

如果改用协程，生产者生产消息后，直接通过yield跳转到消费者开始执行，待消费者执行完毕后，切换回生产者继续生产，效率极高：

	def consumer():
		r = ''
		while True:
			n = yield r
			if not n:
				return
			print('[CONSUMER] Consuming %s...' % n)
			r = '200 OK'

	def produce(c):
		c.send(None)
		n = 0
		while n < 5:
			n = n + 1
			print('[PRODUCER] Producing %s...' % n)
			r = c.send(n)
			print('[PRODUCER] Consumer return: %s' % r)
		c.close()

	c = consumer()
	produce(c)

执行结果：

	[PRODUCER] Producing 1...
	[CONSUMER] Consuming 1...
	[PRODUCER] Consumer return: 200 OK
	[PRODUCER] Producing 2...
	[CONSUMER] Consuming 2...
	[PRODUCER] Consumer return: 200 OK
	[PRODUCER] Producing 3...
	[CONSUMER] Consuming 3...
	[PRODUCER] Consumer return: 200 OK
	[PRODUCER] Producing 4...
	[CONSUMER] Consuming 4...
	[PRODUCER] Consumer return: 200 OK
	[PRODUCER] Producing 5...
	[CONSUMER] Consuming 5...
	[PRODUCER] Consumer return: 200 OK

注意到consumer函数是一个generator，把一个consumer传入produce后：

首先调用c.send(None)启动生成器(php 则不需要)；
然后，一旦生产了东西，通过c.send(n)切换到consumer执行；
consumer通过yield拿到消息，处理，又通过yield把结果传回；

produce拿到consumer处理的结果，继续生产下一条消息；

produce决定不生产了，通过c.close()关闭consumer，整个过程结束。

整个流程无锁，由一个线程执行，produce和consumer协作完成任务，所以称为“协程”，而非线程的抢占式多任务。

最后套用Donald Knuth的一句话总结协程的特点：

“子程序就是协程的一种特例。”

# asyncio
asyncio是Python 3.4版本引入的标准库，直接内置了对异步IO的支持。

asyncio的编程模型就是一个消息循环。我们从asyncio模块中直接获取一个EventLoop的引用，然后把需要执行的协程扔到EventLoop中执行，就实现了异步IO。

用asyncio实现Hello world代码如下：

	import asyncio

	@asyncio.coroutine
	def hello():
	    print("Hello world!")
	    # 异步调用asyncio.sleep(1):
	    r = yield from asyncio.sleep(1)
	    print("Hello again!")

	# 获取EventLoop:
	loop = asyncio.get_event_loop()
	# 执行coroutine
	loop.run_until_complete(hello())
	loop.close()

@asyncio.coroutine把一个generator标记为coroutine类型，然后，我们就把这个coroutine扔到EventLoop中执行。

hello()会首先打印出Hello world!，然后，yield from语法可以让我们方便地调用另一个generator。由于asyncio.sleep()也是一个coroutine，所以线程不会等待asyncio.sleep()，而是直接中断并执行下一个消息循环。当asyncio.sleep()返回时，线程就可以从yield from拿到返回值（此处是None），然后接着执行下一行语句。

把asyncio.sleep(1)看成是一个耗时1秒的IO操作，在此期间，主线程并未等待，而是去执行EventLoop中其他可以执行的coroutine了，因此可以实现并发执行。

我们用Task封装两个coroutine试试：

	import threading
	import asyncio

	@asyncio.coroutine
	def hello():
		print('Hello world! (%s)' % threading.currentThread())
		yield from asyncio.sleep(1)
		print('Hello again! (%s)' % threading.currentThread())

	loop = asyncio.get_event_loop()
	tasks = [hello(), hello()]
	loop.run_until_complete(asyncio.wait(tasks))
	loop.close()

观察执行过程：

	Hello world! (<_MainThread(MainThread, started 140735195337472)>)
	Hello world! (<_MainThread(MainThread, started 140735195337472)>)
	(暂停约1秒)
	Hello again! (<_MainThread(MainThread, started 140735195337472)>)
	Hello again! (<_MainThread(MainThread, started 140735195337472)>)

由打印的当前线程名称可以看出，两个coroutine是由同一个线程并发执行的。

如果把asyncio.sleep()换成真正的IO操作，则多个coroutine就可以由一个线程并发执行。

我们用asyncio的异步网络连接来获取sina、sohu和163的网站首页：

	import asyncio

	@asyncio.coroutine
	def wget(host):
		print('wget %s...' % host)
		connect = asyncio.open_connection(host, 80)
		reader, writer = yield from connect
		header = 'GET / HTTP/1.0\r\nHost: %s\r\n\r\n' % host
		writer.write(header.encode('utf-8'))
		yield from writer.drain()
		while True:
			line = yield from reader.readline()
			if line == b'\r\n':
				break
			print('%s header > %s' % (host, line.decode('utf-8').rstrip()))
		# Ignore the body, close the socket
		writer.close()

	loop = asyncio.get_event_loop()
	tasks = [wget(host) for host in ['www.sina.com.cn', 'www.sohu.com', 'www.163.com']]
	loop.run_until_complete(asyncio.wait(tasks))
	loop.close()

执行结果如下：

	wget www.sohu.com...
	wget www.sina.com.cn...
	wget www.163.com...
	(等待一段时间)
	(打印出sohu的header)
	www.sohu.com header > HTTP/1.1 200 OK
	www.sohu.com header > Content-Type: text/html
	...
	(打印出sina的header)
	www.sina.com.cn header > HTTP/1.1 200 OK
	www.sina.com.cn header > Date: Wed, 20 May 2015 04:56:33 GMT
	...
	(打印出163的header)
	www.163.com header > HTTP/1.0 302 Moved Temporarily
	www.163.com header > Server: Cdn Cache Server V2.0
	...

可见3个连接由一个线程通过coroutine并发完成。

## 小结
1. asyncio提供了完善的异步IO支持；
2. 异步操作需要在coroutine中通过yield from完成；
3. 多个coroutine可以封装成一组Task然后并发执行。
