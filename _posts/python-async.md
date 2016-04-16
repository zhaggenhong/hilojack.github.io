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

消息模型是如何解决同步IO必须等待IO操作这一问题的呢？当遇到IO操作时，代码只负责发出IO请求，不等待IO结果，然后直接结束本轮消息处理，进入下一轮消息处理过程。当IO操作完成后，将收到一条“IO完成”的消息，处理该消息时就可以直接获取IO操作结果。

# TODO
https://docs.python.org/3/library/asyncio-task.html#asyncio.iscoroutinefunction

## BaseEventLoop
https://docs.python.org/3/library/asyncio-eventloop.html#asyncio-hello-world-callback

# yield
it require for the `first send()` to be `None`
> You can't send() a value the first time because the generator did not execute until the point where you have the yield statement, so there is nothing to do with the value.

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
	....

注意到consumer函数是一个generator.

整个流程无锁，由一个线程执行，produce和consumer协作完成任务，所以称为“协程”，而非线程的抢占式多任务。

> Donald Knuth的一句话总结协程的特点： “子程序就是协程的一种特例。”

## send-await
send 会向`yield` 传值，遇到下一个yield 停止
send 遇到`await coroutine()`不会停止，而是`step into coroutine`. 注意`await generator()`语法是非法的

	@types.coroutine
	def switch():
		print('switch')
		a=(yield 1)
		print("return %d", a);
		a=(yield 2)
		print("return %d", a);

	async def coro1():
		print("C1: Start")
		await switch()
		print("C1: Stop")

	async def coro2():
		print("C2: Start")
		print("C2: a")
		print("C2: b")
		print("C2: c")
		print("C2: Stop")

	c1 = coro1()
	c2 = coro2()
	def run(coros):
		coros = list(coros)

		while coros:
			print('start')
			# Duplicate list for iteration so we can remove from original list.
			for coro in list(coros):
				try:
					print(coro.send(None))
					#c1 = coro
				except StopIteration:
					coros.remove(coro)
	run([c1,c2])

# run_until_complete
BaseEventLoop.run_until_complete(future)

	Run until the Future is done.

	If the argument is a coroutine object, it is wrapped by ensure_future().

	Return the Future’s result, or raise its exception.

Example

	import asyncio
	from aiohttp import web

	async def index(request):
		await asyncio.sleep(0.5)
		return web.Response(body=b'<h1>Index</h1>')

	async def hello(request):
		await asyncio.sleep(0.5)
		text = '<h1>hello, %s!</h1>' % request.match_info['name']
		return web.Response(body=text.encode('utf-8'))

	async def init(loop):
		app = web.Application(loop=loop)
		app.router.add_route('GET', '/', index)
		app.router.add_route('GET', '/hello/{name}', hello)
		srv = await asyncio.sleep(5)
		print('Server started at http://127.0.0.1:8000...')
		return srv

	loop = asyncio.get_event_loop()
	i = init(loop)
	loop.run_until_complete(i)
	loop.run_forever()
	print('abc');

# 协程
假设由协程执行，在执行A的过程中，可以随时中断，去执行B，B也可能在执行过程中中断再去执行A.
看起来A、B的执行有点像多线程，但协程的特点在于是一个线程执行，那和多线程比，协程有何优势？

1. 最大的优势就是协程极高的执行效率。因为`子程序切换不是线程切换，而是由程序自身控制`，因此，线程数量越多，协程的性能优势就越明显。
2. 第二大优势就是`不需要多线程的锁机制`，因为只有一个线程，也不存在同时写变量冲突，在协程中控制共享资源不加锁，只需要判断状态就好了，所以执行效率比多线程高很多。

因为协程是一个线程执行，那怎么利用多核CPU呢？最简单的方法是多进程+协程，既充分利用多核，又充分发挥协程的高效率，可获得极高的性能。

## Python对协程的支持有3种实现

1. generator: 我们不但可以通过for循环来迭代，还可以不断调用next()函数获取由yield语句返回的下一个值; send()
2. generator-based:  @asyncio.coroutine + yield from
3. async def+ await

## coroutine vs generator
The word “coroutine”, like the word “generator”, is used for two different (though related) concepts:

1. A `coroutine function`
	(`async def` or `@asyncio.coroutine`). (`iscoroutinefunction()` returns True).
2. A `coroutine object`
	represents a computation or an I/O operation (usually a combination) that will complete eventually.
	A coroutine object (`iscoroutine()` returns True).

### Things a coroutine can do:

1. `result = await future` or `result = yield from future` – suspends the coroutine until the `future` is done, then returns the future’s result, or raises an exception, which will be propagated.
(If the future is cancelled, it will raise a `CancelledError exception`.)
Note that tasks are futures, and everything said about futures also applies to tasks.

2. `result = await coroutine` or `result = yield from coroutine` – wait for another coroutine to produce a result (or raise an exception, which will be propagated). The coroutine expression must be a call to another coroutine.
3. `return expression` – produce a result to the coroutine that is waiting for this one using await or yield from.
4. `raise exception` – raise an exception in the coroutine that is waiting for this one using await or yield from.

### *Run coroutine*:
Calling a coroutine does not start its code running – the coroutine object returned by the call doesn’t do anything until:

1. call `await coroutine or yield from coroutine` from another coroutine (assuming the other coroutine is already running!),
2. or `schedule` its execution using the `ensure_future()` function or the `BaseEventLoop.create_task()` method.

Coroutines (and tasks) can only run when the event loop is running.

# @asyncio.coroutine
Decorator to mark `generator-based coroutines`.
This enables the generator use `yield from` to call `async def coroutines`, and also enables the generator to be called by `async def coroutines`, for instance using an `await` expression.

There is no need to decorate async def coroutines themselves.

> If the generator is not `yielded from` before it is `destroyed`, an error message is logged. See `Detect coroutines never scheduled`.

## call_soon for callback
Example of coroutine displaying "Hello World":

	import asyncio

	async def hello_world():
	    print("Hello World!")

	loop = asyncio.get_event_loop()
	# Blocking call which returns when the hello_world() coroutine is done
	loop.run_until_complete(hello_world())
	loop.close()

See also The Hello World with `call_soon()` example uses the `BaseEventLoop.call_soon()` method to schedule a callback.

> `async def` 定义的new func 属于特殊的generator:
	c.send(None) 才执行:
		且遇到`await` 时停止
		遇不到`await` 时, assert self.done(), "yield from wasn't used with future"
	不执行时：
		RuntimeWarning: coroutine 'compute' was never awaited

Example using the `BaseEventLoop.call_soon()` method to schedule a callback. The callback displays "Hello World" and then stops the event loop:

	import asyncio

	def hello_world(loop):
	    print('Hello World')
	    loop.stop()

	loop = asyncio.get_event_loop()

	# Schedule a call to hello_world()
	loop.call_soon(hello_world, loop)

	# Blocking call interrupted by loop.stop(), otherwise it will not break(forever)
	loop.run_forever()
	loop.close()

### BaseEventLoop.call_soon(callback, *args)

	BaseEventLoop.call_soon(callback, *args)

Arrange for a callback to be called as soon as possible.

1. The callback is called after `call_soon()` returns, when `control returns to the event loop(run_until_complete or run_forever)`.

2. This operates as a `FIFO queue`, callbacks are called in the order in which they are registered. Each callback will be called `exactly once`.

3. Any `positional arguments` after the callback will be passed to the callback when it is called.

4. An instance of `asyncio.Handle` is returned, which can be used to cancel the callback.

## call_later for reschedule
Example of coroutine displaying the current date every second during 5 seconds using the sleep() function:

	import asyncio
	import datetime

	async def display_date(loop):
	    end_time = loop.time() + 5.0
	    while True:
	        print(datetime.datetime.now())
	        if (loop.time() + 1.0) >= end_time:
	            break
	        await asyncio.sleep(1)

	loop = asyncio.get_event_loop()
	# Blocking call which returns when the display_date() coroutine is done
	loop.run_until_complete(display_date(loop))
	loop.close()

The same coroutine implemented using a generator:

	@asyncio.coroutine
	def display_date(loop):
		end_time = loop.time() + 5.0
		while True:
			print(datetime.datetime.now())
			if (loop.time() + 1.0) >= end_time:
				break
			yield from asyncio.sleep(1)

### BaseEventLoop.call_later(delay, callback, *args)
Arrange for the callback to be called after the given delay seconds (either an int or float).

	BaseEventLoop.call_later(delay, callback, *args)
	await asyncio.sleep(delay)

The callback uses the BaseEventLoop.call_later() method to reschedule itself during 5 seconds, and then stops the event loop:

	import asyncio
	import datetime

	def display_date(end_time, loop):
	    print(datetime.datetime.now())
	    if (loop.time() + 1.0) < end_time:
	        loop.call_later(1, display_date, end_time, loop)
	    else:
	        loop.stop()

	loop = asyncio.get_event_loop()

	# Schedule the first call to display_date()
	end_time = loop.time() + 5.0
	loop.call_soon(display_date, end_time, loop)

	# Blocking call interrupted by loop.stop()
	loop.run_forever()
	loop.close()

### 小结
run_until_complete 调用的是: async def func(), 它会等待await 返回结果

call_soon 调用的是func
call_later(1, func, arg.....) 延迟调用func: 和 `await asyncio.sleep(delay)` `await io`相关
run_forever 则会执行上面的func, 如果func 自己不`loop.stop`, 则loop 不会停止

## Chain coroutines
Example chaining coroutines:

	import asyncio

	async def compute(x, y):
		print("Compute %s + %s ..." % (x, y))
		await asyncio.sleep(1.0)
		return x + y

	async def print_sum(x, y):
		result = await compute(x, y)
		print("%s + %s = %s" % (x, y, result))

	loop = asyncio.get_event_loop()
	loop.run_until_complete(print_sum(1, 2))
	loop.close()

`compute()` is chained to `print_sum()`: print_sum() coroutine waits until `compute()` is completed before returning its result.


	![python-async-1.png](/img/python-async-1.png)

The “Task” is created by the BaseEventLoop.run_until_complete() method when it gets a coroutine object instead of a task.

The diagram does not describe exactly how things work internally.
For example, the sleep coroutine creates an `internal future` which uses `BaseEventLoop.call_later()` to wake up the task in 1 second.

# Future

## Future with run_until_complete()
Example combining a Future and a coroutine function:

	import asyncio

	@asyncio.coroutine
	def slow_operation(future):
	    yield from asyncio.sleep(1)
	    future.set_result('Future is done!')

	loop = asyncio.get_event_loop()
	future = asyncio.Future()
	asyncio.ensure_future(slow_operation(future))
	loop.run_until_complete(future)
	print(future.result())
	loop.close()

The coroutine function is responsible for the computation (which takes 1 second) and it stores the result into the future.
The `run_until_complete()` method waits for the completion of the future.

Note The `loop.run_until_complete()` method uses internally the `Future.add_done_callback()` method to be `notified` when the future is done.

### Future.add_done_callback
Add a callback to be run when the future becomes done.

	add_done_callback(fn)

The callback is called with a single argument - `the future object`.
If the future is already done when this is called, the callback is scheduled with `call_soon()`.

## Future with run_forever()
The previous example can be written differently using the `Future.add_done_callback()` method to describe explicitly the control flow:

	import asyncio

	@asyncio.coroutine
	def slow_operation(future):
	    yield from asyncio.sleep(1)
	    future.set_result('Future is done!')

	def got_result(future):
	    print(future.result())
	    loop.stop()

	loop = asyncio.get_event_loop()
	future = asyncio.Future()
	asyncio.ensure_future(slow_operation(future))
	future.add_done_callback(got_result)
	try:
	    loop.run_forever()
	finally:
	    loop.close()

In this example, the future is used to link `slow_operation()` to `got_result()`:
	when `slow_operation()` is done, `got_result()` is called with the result.

# run_until_complete and run_forever

	loop.run_until_complete(coroutine)

	loop.call_soon(display_date, end_time, loop)
	loop.call_later(delay, callback, *args)
	loop.run_forever()

Future 是一个callback for coroutine

	future = asyncio.Future()
	asyncio.ensure_future(coroutine(future)); 加任务
	future.add_done_callback(got_result)
	    loop.run_forever()

Parallel

	tasks = [
		asyncio.ensure_future(factorial("A", 2)),
		asyncio.ensure_future(factorial("B", 3)),
		asyncio.ensure_future(factorial("C", 4))]
	loop.run_until_complete(asyncio.wait(tasks))

# Tasks

## Parallel execution of tasks
Example executing 3 tasks (A, B, C) in parallel:

	import asyncio
	async def factorial1(name, number):
		f = 1
		for i in range(2, number+1):
			print("Task %s: Compute factorial(%s)..." % (name, i))
			await asyncio.sleep(1)
			f *= i
		print("Task %s: factorial(%s) = %s" % (name, number, f))

	async def factorial(name, number):
		print('start task:%s' % name)
		await factorial1(name, number)
		print('finished task:%s' % name)

	loop = asyncio.get_event_loop()
	tasks = [
		asyncio.ensure_future(factorial("A", 2)),
		asyncio.ensure_future(factorial("B", 3)),
		asyncio.ensure_future(factorial("C", 4))]
	loop.run_until_complete(asyncio.wait(tasks))
	loop.close()

Output:

	➜ > test p3 a.py
	start task:A
	Task A: Compute factorial(2)...
	start task:B
	Task B: Compute factorial(2)...
	start task:C
	Task C: Compute factorial(2)...
	Task A: factorial(2) = 2
	finished task:A
	Task B: Compute factorial(3)...
	Task C: Compute factorial(3)...
	Task B: factorial(3) = 6
	finished task:B
	Task C: Compute factorial(4)...
	Task C: factorial(4) = 24
	finished task:C

asyncio.wait 会自动用`asyncio.ensure_future` wrap `coroutine`, 所以可以简写成：

	tasks = [
		factorial("A", 2),
		factorial("B", 3),
		factorial("C", 4)]
	loop.run_until_complete(asyncio.wait(tasks))

# asyncio
asyncio是Python 3.4版本引入的标准库，直接内置了对异步IO的支持。

asyncio的编程模型就是一个消息循环。我们从asyncio模块中直接获取一个EventLoop的引用，然后把需要执行的协程扔到EventLoop中执行，就实现了异步IO。

用asyncio 实现Hello world 代码如下：

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

`@asyncio.coroutine`把一个`generator`标记为`coroutine`类型，然后，我们就把这个`coroutine`扔到EventLoop中执行。(看起来不是必须的)

hello()会首先打印出Hello world!，然后，yield from语法可以让我们方便地调用另一个generator。
由于asyncio.sleep()也是一个coroutine，所以线程不会等待asyncio.sleep()，而是直接中断并执行下一个消息循环。当asyncio.sleep()返回时，线程就可以从yield from拿到返回值（此处是None），然后接着执行下一行语句。

把asyncio.sleep(1)看成是一个耗时1秒的IO操作，在此期间，主线程并未等待，而是去执行EventLoop中其他可以执行的coroutine了，因此可以实现并发执行。

## asyncio coroutine example 1
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

## asyncio coroutine example 2
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

# async/await
用asyncio 提供的`@asyncio.coroutine` 可以把一个generator标记为coroutine类型，然后在coroutine内部用yield from调用另一个coroutine实现异步操作。

为了简化并更好地标识异步IO，从Python 3.5开始引入了新的语法async和await，可以让coroutine的代码更简洁易读。

请注意，async和await是针对coroutine的新语法，要使用新的语法，只需要做两步简单的替换：

1. 把`@asyncio.coroutine`替换为`async`: 将generator 变成 coroutine；
2. 把`yield from`替换为`await`: 后面接一个子例程 sub coroutine, 会被加入到`asyncio.get_event_loop`，
	loop 会循环侦测状态并执行(没有await 的coroutine 不会被执行)
	`a = await func()` 其实是异步的`yield func()`

让我们对比一下上一节的代码：

	@asyncio.coroutine
	def hello():
		print("Hello world!")
		r = yield from asyncio.sleep(1)
		print("Hello again!")

用新语法重新编写如下：

	async def hello():
		print("Hello world!")
		r = await asyncio.sleep(1)
		print("Hello again!")

> Python从3.5版本开始为asyncio提供了async和await的新语法；

# aiohttp
> 如果把asyncio用在服务器端，例如Web服务器，由于HTTP连接就是IO操作，因此可以用单线程+coroutine实现多用户的高并发支持。
> asyncio实现了TCP、UDP、SSL等协议，aiohttp则是基于asyncio实现的HTTP框架。

我们先安装aiohttp：

	pip3 install aiohttp

然后编写一个HTTP服务器，分别处理以下URL：

	/ - 首页返回b'<h1>Index</h1>'；
	/hello/{name} - 根据URL参数返回文本hello, %s!。

代码如下：

	import asyncio
	from aiohttp import web

	async def index(request):
	    await asyncio.sleep(0.5)
	    return web.Response(body=b'<h1>Index</h1>')

	async def hello(request):
	    await asyncio.sleep(0.5)
	    text = '<h1>hello, %s!</h1>' % request.match_info['name']
	    return web.Response(body=text.encode('utf-8'))

	async def init(loop):
	    app = web.Application(loop=loop)
	    app.router.add_route('GET', '/', index)
	    app.router.add_route('GET', '/hello/{name}', hello)
	    srv = await loop.create_server(app.make_handler(), '127.0.0.1', 8000)
	    print('Server started at http://127.0.0.1:8000...')
	    return srv

	loop = asyncio.get_event_loop()
	loop.run_until_complete(init(loop))
	loop.run_forever()

注意aiohttp的初始化函数init()也是一个coroutine，`loop.create_server()` 则利用asyncio创建TCP服务。

> https://docs.python.org/3/library/asyncio-eventloop.html

- run_until_complete

	vim /usr/local/Cellar/python3/3.5.0/Frameworks/Python.framework/Versions/3.5/lib/python3.5/asyncio/base_events.py +291

# Other

## Set signal handlers for SIGINT and SIGTERM
Register handlers for signals SIGINT and SIGTERM using the `BaseEventLoop.add_signal_handler()` method:

	import asyncio
	import functools
	import os
	import signal

	def ask_exit(signame):
		print("got signal %s: exit" % signame)
		loop.stop()

	loop = asyncio.get_event_loop()
	for signame in ('SIGINT', 'SIGTERM'):
		loop.add_signal_handler(getattr(signal, signame),
								functools.partial(ask_exit, signame))

	print("Event loop running forever, press Ctrl+C to interrupt.")
	print("pid %s: send SIGINT or SIGTERM to exit." % os.getpid())
	try:
		loop.run_forever()
	finally:
		loop.close()
