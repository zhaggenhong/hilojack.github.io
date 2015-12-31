---
layout: page
title:
category: blog
description:
---
# Preface


# Data Type
数据类型

- String
- List( Array)
- None

Example

	type(range(1)) is list
	type(9.0) is float

Convert data type:

	int('07')
	float(9)

## type

	>>> type(fn)==types.FunctionType
	True
	>>> type(className); types.type
	>>> type(obj); class className
	>>> type(abs)==types.BuiltinFunctionType
	True
	>>> type(lambda x: x)==types.LambdaType
	True
	>>> type((x for x in range(10)))==types.GeneratorType
	True

## isinstance
也可用来判断数据类型

	isinstance('abc', str); # True
	isinstance(b'abc', bytes); # True
	isinstance(1, int); # True
	isinstance('abc', Iterable); # True
	isinstance([1, 2, 3], (list, tuple))

# Bytes

	>>> type(b'abc')
	<class 'bytes'>
	>>> type('abc')
	<class 'str'>

	>>> type(b'abc'[2])
	<class 'int'>

# String
double quotes and single quotes is same

	print "a\nb" ;# The character here "\n" is new line
	print 'a\nb'

## Access String
like list

	'hilojack'[4:]
	'hilo' in 'hilojack'

## string func

	## substring count
	'hilo hilo'.count('hilo')

	## capitalize
	'hilo'.capitalize()
	'hilo'.upper()
	'hilo'.lower()

	## len
	len('abc')

	## split
	'ab cd'.split(' ') # ['ab', 'cd']

	## sorted
	sorted('a zx') # [' ', 'a', 'x', 'z']

## search replace

	'hilojack'.find('jack');//4
	'hilo' in 'hilojack'
	str.replace(needle, word, 1); //replace the first needle with word

## pad
zfill

	print '1'.zfill(2);

## trim

	'a\n  '.strip()+',end'

## Concat String

	>>> print 'a'+'b'+'c'	# with no space
	abc	# "abc\n"
	>>> print 'a' 'b' 	'c'   # with no space
	>>> print 'a''b''c'   # with no space
	abc # "abc\n"
	>>> print var1 var2   # syntax error
	>>> print '-'*6
	------
	>>> print '-' * 6
	------

long delimiter `"""` and `'''`(same): `\n` is still transfered by python

	print """
	a\nbc
	"""
	print """ab\nc"""
	print '''ab\nc'''

### Escape Sequences

	\n	ASCII linefeed (LF)
	\N{name}	Character named name in the Unicode database (Unicode only)
	\r ASCII	Carriage Return (CR)
	\uxxxx	Character with 16-bit hex value xxxx (Unicode only)
	\Uxxxxxxxx	Character with 32-bit hex value xxxxxxxx (Unicode only)
	\v	ASCII vertical tab (VT)
	\ooo	Character with octal value ooo
	\xhh	Character with hex value hh

### print

	>>> print 'a', 'b', 'c' # with space and new line
	a b c	# "a b c\n"
	>>> print 'a', 'b', 'c',# with space only
	a b c	# "a b c "

with no space and new line:

	>> sys.stdout.write('string')
	string

## Format String

	>>> format='%s'
	>>> print format % 'part1' 'part2'
	part1part2

	>>> print 'This is (%r) (%s)' % ("Hilojack\"", "Blog")
	This is ('Hilojack"') (Blog)
	>>> print '''This is (%r) (%s)''' % ("Hilojack\"", "Blog")
	This is ('Hilojack"') (Blog)

> %r displays the "raw" data

## string like list

	print "abc"[-1]
	for in "abc"

# Dict

	dic = {'x': 1, 'y': 2}
	del dict['x']
	dict[1]=123
	dict.setdefault(1, 123)

## access dict

	dict['x']
	dict.get('x', 'not found')
	dict.items()
		[('x', 1), ('y', 2)]

## dict keys and values

	dict.keys();//keys list
	dict.values();//values list

delete key

	dict.pop(key)

## foreach dict

	for k,v abbrev in dict.items():
		print "%s : %d" % (k,v)

	>>>for k,v in {'k1':'v1','k2':'v2'}.items(): print k,v;
	...
	k2 v2
	k1 v1
	>>> for v in {'k1':'v1','k2':'v2'}.items(): print v;
	...
	('k2', 'v2')
	('k1', 'v1')

list and enumerate

	>>> for k,v in [['k1','v1'],['k2','v2']]: print k,v
	...
	k1 v1
	k2 v2
	>>> for k,v in [('k1','v1'),('k2','v2')]: print k,v
	...
	k1 v1
	k2 v2

	>>> for k,v in enumerate([['k1','v1'],['k2','v2']]): print k,v
	...
	0 ['k1', 'v1']
	1 ['k2', 'v2']

# set
set和dict类似，也是一组key的集合，但不存储value。由于key不能重复，所以，在set中，没有重复的key。

要创建一个set，需要提供一个list作为输入集合：

	>>> s = set([1, 2, 3])
	>>> s
	{1, 2, 3}

注意，传入的参数[1, 2, 3]是一个list，而显示的{1, 2, 3}只是告诉你这个set内部有1，2，3这3个元素，显示的顺序也不表示set是有序的。。

重复元素在set中自动被过滤：

	>>> s = set([1, 1, 2, 2, 3, 3])
	>>> s
	{1, 2, 3}

通过add(key)方法可以添加元素到set中，可以重复添加，但不会有效果：

	>>> s.add(4)
	>>> s
	{1, 2, 3, 4}
	>>> s.add(4)
	>>> s
	{1, 2, 3, 4}

通过remove(key)方法可以删除元素：

	>>> s.remove(4)
	>>> s
	{1, 2, 3}

set可以看成数学意义上的无序和无重复元素的集合，因此，两个set可以做数学意义上的交集、并集等操作：

	>>> s1 = set([1, 2, 3])
	>>> s2 = set([2, 3, 4])
	>>> s1 & s2
	{2, 3}
	>>> s1 | s2
	{1, 2, 3, 4}

set和dict的唯一区别仅在于没有存储对应的value，但是，set的原理和dict一样

# list and tuple
因为tuple不可变，所以代码更安全。如果可能，能用tuple代替list就尽量用tuple。

	list = [1,2]
	list +=[3,]

	tuple = (1,2)
	tuple +=(3,)

	sorted(['a', 'z', 'g']).pop(-1); # last
	sorted(['a', 'z', 'g']).pop(); # last
	sorted('azg']).pop(0); # first

> For more details, refer to `pydoc list`

The difference between list and tuple:

	1. tuple can be dictionary identifier key
	{(1,2):1} ok
	{[1,2]:1} error

	2. Tuples are immutable, and usually contain an heterogeneous sequence ..., and list is sorted

	3. tuple can not be accessed by index, so do deleted
	tuple[0] error
	list[0]	ok

	del tuple[0] error

## zip list

	>>> xl = [1,3,5]
	>>> yl = [9,12,13]
	>>> print zip(xl,yl)
	[(1, 9), (3, 12), (5, 13)]

## join and split

	','.join([1,2])
	'1,2'.split(',');

## shuffle list

	import random
	list=[1,2,3]
	random.shuffle(list)

## Access List

	list[0]
	[1,2][0]
	[1,2][-1]

slice

	list[start:end:step]
	list[0:3]
	list[:-1]
	list[::5]
	print list[0:10:2]

## in array

	>>> 0 in range(1)
	True
	>>> x in range(1, 10)
	>>> 'hilo' in 'hilojack'

## print

	print list
	print '%r' % list

## Common list Function


	.index(value, [start, [stop]])
	.count(value) -> integer -- return number of occurrences of value
	.reverse() -> reverse *IN PLACE*
	.sort(cmp=None, key=None, reverse=False) -- stable sort *IN PLACE*;

remove and insert(in place)

	.insert(index, object)
	.remove(value) -- remove first occurrence of value.

## range:

	>>> print range(4)
	[0, 1, 2, 3]
	>>> print range(2,4)
	[2, 3]

## pop,append

	>>> list = [1,2]
	del list[1]
	>>> list.append(3)
	>>> list.pop(-1) # same as list.pop()
	3
	>>> list.pop(0)
	1

## unpack list

	>>> print argv
	[1, 2]
	>>> a,b = [1, 2]

	>>> print aa()
	(1, 2)
	>>> a,b = aa()

## any

	if needle.endswith('ly') or needle.endswith('ed') or
		needle.endswith('ing') or needle.endswith('ers'):
		print('Is valid')
	else:
		print('Invalid')

改成:

	if any([needle.endswith(e) for e in ('ly', 'ed', 'ing', 'ers')]):
		print('Is valid')
	else:
		print('Invalid')

Syntax:

	any([ expression(e) for e in (....)])
	any([True, False, False]);//True

列表解析:

	[ expression(e) for e in (....)])
	[ expression(e) for e in (....) if <condition>])


例如获取100以内的奇数：

	print filter(lambda n: (n%2) == 1, range(100))

当然对于上面的例子，也可以使用列表解析实现：

	print [i for i in range(100) if i%2 == 1]

## map


	# map的func为None 作用同zip()
	print map(None, [4, 5, 6])
	print map(None, [1, 2, 3], [4, 5, 6])

	# map 针对一个序列
	print map(lambda x: x*2, [4, 5, 6])

	# map 针对多个序列
	print map(lambda x, y: x + y, [1, 2, 3], [4, 5, 6])

out:

	[4, 5, 6]
	[(1, 4), (2, 5), (3, 6)]
	[8, 10, 12]
	[5, 7, 9]

## reduce
迭代可以用于累加...

	print reduce(lambda x, y: x + y, range(10))
		0 + 1
		  + 2 + ... + 9
	print reduce(lambda x, y: x + y, range(10), 100)
		100 + 0
		  + 1 + 2 + ... + 9

我们自己也可以实现一个reduce函数：

	def xreduce(bin_func, seq, init=None):
		Iseq = list(seq)
		if init is None:
			res = Iseq.pop(0)
		else:
			res = init
		for obj in Iseq:
			res = bin_func(res,obj)
		return res

# Number

## math

	int('1')
	float('1')
	round(1.7333)
	range(1,10 [,step])

## random

	import random
	random.randint(3,8)

## math operator

	print x**2; # x^2

