---
layout: page
title:
category: blog
description:
---
# Preface


# function
var 默认是global 的

	def func(s1,s2=None):
		global X
		print global_var
		func.count++
		local_var=4
		retrun s1,s2

	s1,s2 = func("-%s-" % 'Hello','-%s-' % 'Hilo' 'jack')
	print s1,s2

output:

	-Hello- -Hilojack-

## static var

	def foo():
		foo.counter += 1
		print "Counter is %d" % foo.counter
	foo.counter = 0

or:

	def foo():
		try:
			foo.counter += 1
		except AttributeError:
			foo.counter = 1

终极:

	def static_vars(**kwargs):
		def decorate(func):
			for k in kwargs:
				setattr(func, k, kwargs[k])
			return func
		return decorate

	@static_vars(counter=0)
	def foo():
		foo.counter += 1
		print "Counter is %d" % foo.counter

## lambda function

	func = lambda x, y: x + y
	func = (lambda x, y: x + y)
	func(1,2)

lambda 不能显式使用return :

	# error
	func = lambda x, y: return x + y

## multi args
可变参数 定义：

	def func(*args):
		s1,s2 = args

使用

	>>> nums = [1, 2, 3]
	>>> calc(nums[0], nums[1], nums[2])
	>>> calc(*nums)

## keyword args
关键字参数

	def person(name, age, **kw):
		print('name:', name, 'age:', age, 'other:', kw)

	>>> extra = {'city': 'Beijing', 'job': 'Engineer'}
	>>> person('Jack', 24, city=extra['city'], job=extra['job'])
	name: Jack age: 24 other: {'city': 'Beijing', 'job': 'Engineer'}

当然，上面复杂的调用可以用简化的写法：

	>>> extra = {'city': 'Beijing', 'job': 'Engineer'}
	>>> person('Jack', 24, **extra)
	name: Jack age: 24 other: {'city': 'Beijing', 'job': 'Engineer'}

## named args
如果要限制关键字参数的名字，就可以用命名关键字参数，例如，只接收city和job作为关键字参数。这种方式定义的函数如下：

	def person(name, age, *, city='bj', job):
		print(name, age, city, job)

`*`后面的参数被视为命名关键字参数, 传参时必须带参数名
`*`前面的参数可带可不带参数名

	>>> person('Jack', 24, job='Engineer')

## 各种参数
定义一个函数，包含上述若干种参数：

	def f1(a, b, c=0, *args, **kw):
		print('a =', a, 'b =', b, 'c =', c, 'args =', args, 'kw =', kw)

	def f2(a, b, c=0, *, d, **kw):
		print('a =', a, 'b =', b, 'c =', c, 'd =', d, 'kw =', kw)

# Condition & Loop

## control

	exit(0);
		like exit(0) in c

	sys.exit()
		SystemExit 异常，没有捕获这个异常，会直接退出；捕获这个异常可以做一些额外的清理工作。


## try catch

	try:
		do sth.
	except EOFError:
		print '\nBye!'
	except ValueError:
		return None
	finally:
		do sth.

## if

	x=10;
	if 1<=x<=5:
		print x
	elif x==6: print x*x
	else: print x*x*x

if is

	if L is None:
	if L == None:

## loop

### for

	for i in range(5): print i
	for i in [1],2,3: print i

Iterator For

	[w.capitalize() for w in ['aa','bb','cc']]

#### for 本质

	for arg in argv:
	for(i=0; i<len(argv); i++): arg = argv[i] //len(argv) 是实时计算的, 会受argv.pop 的影响

### 判断对象是否可迭代
可用：

	hasattr([],'__iter__')

或者:

	>>> from collections import Iterable
	>>> isinstance('abc', Iterable) # str是否可迭代
	True
	>>> isinstance([1,2,3], Iterable) # list是否可迭代
	True
	>>> isinstance(123, Iterable) # 整数是否可迭代
	False

Python内置的enumerate函数可以把一个list变成索引-元素对，这样就可以在for循环中同时迭代索引和元素本身：

	>>> for i, value in enumerate(['A', 'B', 'C']):
	...     print(i, value)
	...
	0 A
	1 B

### while

	while x<6:
		Statement

# 三元运算符
在python中的格式为

	为真时的结果 if 判定条件 else 为假时的结果

	1 if 5>3 else 0

# List Comprehensions, 列表生成式
列表生成式即List Comprehensions

	>>> [x * x for x in range(1, 11)]
	[1, 4, 9, 16, 25, 36, 49, 64, 81, 100]

for循环后面还可以加上if判断，这样我们就可以筛选出仅偶数的平方：

	>>> [x * x for x in range(1, 11) if x % 2 == 0]
	[4, 16, 36, 64, 100]

还可以使用两层循环，可以生成全排列：

	>>> [m + n for m in 'AB' for n in 'XYZ']
	['AX', 'AY', 'AZ', 'BX', 'BY', 'BZ']

列出文件

	>>> import os # 导入os模块，模块的概念后面讲到
	>>> [d for d in os.listdir('.')] # os.listdir可以列出文件和目录

# generator, 生成器
通过列表生成式，直接创建一个大列表, 会受到内存限制。我们可以使用generator. 只需要将list `[]` 改成`()`

	g = (x * x for x in range(10))
	>>> next(g)
	0
	>>> next(g)
	1
	>>> next(g)
	4
	>> [for i in g]

斐波拉契数列用列表生成式写不出来，但是，用函数把它打印出来却很容易：

	def fib(max):
		n, a, b = 0, 0, 1
		while n < max:
			print(b)
			a, b = b, a + b
			n = n + 1
		return 'done'

上面的函数可以输出斐波那契数列的前N个数, 改成generator 就是：

	def fib(max):
		n, a, b = 0, 0, 1
		while n < max:
			yield b
			a, b = b, a + b
			n = n + 1
		return 'done'

# Iterator, 迭代器
生成器(generator)都是Iterator对象，但list、dict、str虽然是Iterable，却不是Iterator。

把list、dict、str等Iterable变成Iterator可以使用iter()函数：

	>>> import collections
	>>> isinstance(iter([]), collections.Iterator)
	True
	>>> isinstance(iter('abc'), collections.Iterator)
	True
	>>> isinstance([i for i in [1,2]], collections.Iterator)
	False

为什么list、dict、str等数据类型不是Iterator？

这是因为Python的Iterator对象表示的是一个数据流，Iterator对象可以被next()函数调用并不断返回下一个数据，直到没有数据时抛出StopIteration错误。可以把这个数据流看做是一个有序序列，但我们却不能提前知道序列的长度，只能不断通过next()函数实现按需计算下一个数据，所以Iterator的计算是惰性的，只有在需要返回下一个数据时它才会计算。

Iterator甚至可以表示一个无限大的数据流，例如全体自然数。而使用list是永远不可能存储全体自然数的

1. 凡是可作用于for循环的对象都是Iterable类型；
2. 凡是可作用于next()函数的对象都是Iterator类型，它们表示一个惰性计算的序列；

Python的for循环本质上就是通过不断调用next()函数实现的，例如：

	for x in [1, 2, 3, 4, 5]:
		pass

实际上完全等价于：

	it = iter([1, 2, 3, 4, 5])
	while True:
		try:
			x = next(it)
		except StopIteration:
			break

# logic expression

	>>> x =5; 1 < x < 10
	True
	>>> False or 0 or '' or 3
	3

	and or not
	!= (not equal)
	== (equal)
	>= (greater-than-equal)
	<= (less-than-equal)
	True
	False

# with
with 可以捕获异常, 类必须支持`__enter__, __exit__`:

	class DummyResource:
		def __enter__(self):
			print('enter');
			return self	  # 可以返回不同的对象
		def __exit__(self, exc_type, exc_value, exc_tb):
			print( 'Free resource.')
			if exc_tb is None:
				print('Exited without exception.')
			else:
				print('Exited with exception raised.')
				return True   # 不再raise 出异常

	with DummyResource() as obj:
		10/0
		print('do sth...')

利用with 断言指定类型的Error，比如通过d['empty']访问不存在的key时，断言会抛出KeyError：

	with self.assertRaises(KeyError):
		value = d['empty']
