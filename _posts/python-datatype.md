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

# list and tuple

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

	list[start:end:step]
	list[0:3]
	list[:-1]
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
	.insert(index, object)
	.remove(value) -- remove first occurrence of value.
	.count(value) -> integer -- return number of occurrences of value
	.reverse() -> reverse *IN PLACE*
	.sort(cmp=None, key=None, reverse=False) -- stable sort *IN PLACE*;

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

