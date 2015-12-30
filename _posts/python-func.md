---
layout: page
title:
category: blog
description:
---
# Preface

函数式编程

# map reduce
> http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/0014317852443934a86aa5bb5ea47fbbd5f35282b331335000

Python内建了map()和reduce()函数。

如果你读过Google的那篇大名鼎鼎的论文“MapReduce: Simplified Data Processing on Large Clusters”，你就能大概明白map/reduce的概念。

## map
我们先看map。map()函数接收两个参数，一个是函数，一个是Iterable，map将传入的函数依次作用到序列的每个元素，并把结果作为新的Iterator返回。

	>>> def f(x):
	...     return x * x
	...
	>>> r = map(f, [1, 2, 3, 4, 5, 6, 7, 8, 9])
	>>> list(r)
	[1, 4, 9, 16, 25, 36, 49, 64, 81]

map()传入的第一个参数是f，即函数对象本身。由于结果r是一个Iterator，Iterator是惰性序列，因此通过list()函数让它把整个序列都计算出来并返回一个list。
map()作为高阶函数，事实上它把运算规则抽象了。比如，把这个list所有数字转为字符串：

	>>> list(map(str, [1, 2, 3, 4, 5, 6, 7, 8, 9]))
	['1', '2', '3', '4', '5', '6', '7', '8', '9']

## reduce
reduce把一个函数作用在一个序列[x1, x2, x3, ...]上，这个函数必须接收两个参数，reduce把结果继续和序列的下一个元素做累积计算，其效果就是：

	reduce(f, [x1, x2, x3, x4]) = f(f(f(x1, x2), x3), x4)

比方说对一个序列求和，就可以用reduce实现：

	>>> from functools import reduce
	>>> def add(x, y):
	...     return x + y
	...
	>>> reduce(add, [1, 3, 5, 7, 9])

我来利用map+reduce 实现 str2int的函数就是：

	from functools import reduce

	def str2int(s):
		def fn(x, y):
			return x * 10 + y
		def char2num(s):
			return {'0': 0, '1': 1, '2': 2, '3': 3, '4': 4, '5': 5, '6': 6, '7': 7, '8': 8, '9': 9}[s]
		return reduce(fn, map(char2num, s))

还可以用lambda函数进一步简化成：

	from functools import reduce
	def char2num(s):
		return {'0': 0, '1': 1, '2': 2, '3': 3, '4': 4, '5': 5, '6': 6, '7': 7, '8': 8, '9': 9}[s]

	def str2int(s):
		return reduce(lambda x, y: x * 10 + y, map(char2num, s))

# filter
filter 返回的也是惰性Iterator

	list(filter(lambda x:x%2==0, [1,2,3]));

## 用filter求素数

计算素数的一个方法是埃氏筛法，它的算法理解起来非常简单：

	首先，列出从2开始的所有自然数，构造一个序列：

	2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, ...

	取序列的第一个数2，它一定是素数，然后用2把序列的2的倍数筛掉：

	3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, ...

	取新序列的第一个数3，它一定是素数，然后用3把序列的3的倍数筛掉：

	5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, ...

	取新序列的第一个数5，然后用5把序列的5的倍数筛掉：

	7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, ...

	不断筛下去，就可以得到所有的素数。

用Python来实现这个算法，可以先构造一个从3开始的奇数序列：

	def _odd_iter():
		n = 1
		while True:
			n = n + 2
			yield n

注意这是一个生成器，并且是一个无限序列。 然后定义一个筛选函数：

	def _not_divisible(n):
		return lambda x: x % n > 0

最后，定义一个生成器，不断返回下一个素数：

	def primes():
		yield 2
		it = _odd_iter() # 初始序列
		while True:
			n = next(it) # 返回序列的第一个数
			yield n
			it = filter(_not_divisible(n), it) # 构造新序列

这个生成器先返回第一个素数2，然后，利用filter()不断产生筛选后的新的序列。

打印1000以内的素数:

	for n in primes():
		if n < 1000:
			print(n)
		else:
			break

## 回数生成器

	filter(lambda n:str(n)[::-1]==str(n), range(10,99))

# sorted
ython内置的sorted()函数就可以对list进行排序：

	>>> sorted([36, 5, -12, 9, -21])
	[-21, -12, 5, 9, 36]

此外，sorted()函数也是一个高阶函数，它还可以接收一个key函数来实现自定义的排序，例如按绝对值大小排序：

	>>> sorted([36, 5, -12, 9, -21], key=abs)
	[5, 9, -12, -21, 36]

现忽略大小写的排序：

	>>> sorted(['bob', 'about', 'Zoo', 'Credit'], key=str.lower)
	['about', 'bob', 'Credit', 'Zoo']

要进行反向排序，不必改动key函数，可以传入第三个参数reverse=True：

	>>> sorted(['bob', 'about', 'Zoo', 'Credit'], key=str.lower, reverse=True)
	['Zoo', 'Credit', 'bob', 'about']

# 闭包返回函数
我们来看一个例子：

	def count():
		fs = []
		for i in range(1, 4):
			def f():
				 return i*i
			fs.append(f)
		return fs

	f1, f2, f3 = count()

在上面的例子中，每次循环，都创建了一个新的函数，然后，把创建的3个函数都返回了。

你可能认为调用f1()，f2()和f3()结果应该是1，4，9，但实际结果是：

	>>> f1()
	9
	>>> f2()
	9
	>>> f3()
	9

全部都是9！原因就在于返回的函数引用了变量i，但它并非立刻执行。等到3个函数都返回时，它们所引用的变量i已经变成了3，因此最终结果为9。

返回闭包时牢记的一点就是：返回函数不要引用任何循环变量，或者后续会发生变化的变量。

如果一定要引用循环变量怎么办？方法是再创建一个函数，用该函数的参数绑定循环变量当前的值，无论该循环变量后续如何更改，已绑定到函数参数的值不变：

	def count():
		def f(j):
			def g():
				return j*j
			return g
		fs = []
		for i in range(1, 4):
			fs.append(f(i)) # f(i)立刻被执行，因此i的当前值被传入f()
		return fs

# decorator, 装饰器
由于函数也是一个对象，而且函数对象可以被赋值给变量，所以，通过变量也能调用该函数。

	>>> def now():
	...     print('2015-3-25')
	...
	>>> f = now
	>>> f()
	2015-3-25

函数对象有一个__name__属性，可以拿到函数的名字：

	>>> now.__name__
	'now'
	>>> f.__name__
	'now'

现在，假设我们要增强now()函数的功能，比如，在函数调用前后自动打印日志，但又不希望修改now()函数的定义，这种在代码运行期间动态增加功能的方式，称之为“装饰器”（Decorator）。

本质上，decorator就是一个返回函数的高阶函数。所以，我们要定义一个能打印日志的decorator，可以定义如下：

	def log(func):
		def wrapper(*args, **kw):
			print('call %s():' % func.__name__)
			return func(*args, **kw)
		return wrapper

我们要借助Python的@语法，把decorator置于函数的定义处：

	@log
	def now():
		print('2015-3-25')

调用now()函数，不仅会运行now()函数本身，还会在运行now()函数前打印一行日志：

	>>> now()
	call now():
	2015-3-25

如果decorator本身需要传入参数，那就需要编写一个返回decorator的高阶函数，写出来会更复杂。比如，要自定义log的文本：

	def log(text):
		def decorator(func):
			def wrapper(*args, **kw):
				print('%s %s():' % (text, func.__name__))
				return func(*args, **kw)
			return wrapper
		return decorator

这个3层嵌套的decorator用法如下：

	@log('execute')
	def now():
		print('2015-3-25')

	log('excute')(now)

## __name__
经过decorator装饰之后的函数，它们的__name__已经从原来的'now'变成了'wrapper'：

	>>> now.__name__
	'wrapper'

因为返回的那个wrapper()函数名字就是'wrapper'，所以，需要把原始函数的__name__等属性复制到wrapper()函数中，否则，有些依赖函数签名的代码执行就会出错。

不需要编写`wrapper.__name__ = func.__name__`这样的代码，Python内置的`functools.wraps`就是干这个事的，所以，一个完整的decorator的写法如下：

	import functools

	def log(func):
		@functools.wraps(func)
		def wrapper(*args, **kw):
			print('call %s():' % func.__name__)
			return func(*args, **kw)
		return wrapper

或者针对带参数的decorator：

	import functools

	def log(text):
		def decorator(func):
			@functools.wraps(func)
			def wrapper(*args, **kw):
				print('%s %s():' % (text, func.__name__))
				return func(*args, **kw)
			return wrapper
		return decorator

# Partial function,偏函数
用于定制函数

	def int2(x, base=2):
		return int(x, base)

functools.partial就是帮助我们创建一个偏函数的，不需要我们自己定义int2()，可以直接使用下面的代码创建一个新的函数int2：

	>>> import functools
	>>> int2 = functools.partial(int, base=2)
	>>> int2('1000000')
	64
	>>> int2('1010101')
	85

最后，创建偏函数时，实际上可以接收函数对象、*args和**kw这3个参数，当传入：

	max2 = functools.partial(max, 10)

实际上会把10作为*args的一部分自动加到左边，也就是：

	max2(5, 6, 7)

相当于：

	args = (10, 5, 6, 7)
	max(*args)
