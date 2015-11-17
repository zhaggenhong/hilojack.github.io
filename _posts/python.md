---
layout: page
title:	Learn Python
category: blog
description: 
---
# Preface

# todo
to read
http://learnpythonthehardway.org/book/ex25.html

# python3 vs python2
http://www.dantangfan.com/diff-py2-py3/
官方提供：
https://pythonhosted.org/six/

Please use Python3
1. http://asmeurer.github.io/python3-presentation/slides.html#1

# todo python3
begin:
http://docspy3zh.readthedocs.org/en/latest/tutorial/index.html
deep: 
http://woodpecker.org.cn/diveintopython3/

# Help

	$ pydoc <name>
	$ pydoc pydoc
	$ pydoc open
	$ pydoc file

## in pythoe

	> help(file)
	var=1
	> help(var)

# Shell 

	cat a.py | python
	cat a.py | python - arg1

Note: raw_input 将不会正常工作，We better use :

	python <(cat p.py)
	python p.py
	

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

	1. tuple can be dictionary identifier
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

# IO

## url

	from urllib import urlopen

readline:

	urlp = urlopen(url);
	while(line = urlp.readline()){
	
	}

readlines:

	for line in urlopen(url).readlines():
		print line.strip()

## argv

	from sys import argv # import module "sys" and objects: argv
	script, arg1, arg2 = argv

	import sys;
	sys.argv

## input

	str = raw_input()
	str = raw_input("Input some string:")

## Directory

	from os.path import exists
	exists(file)

## File

	print open('a.php').read(); //cat a.php

### Open File

	fp = open(filename[, mode])
	mode:
		'r' 'w' 'a'
		append a '+' to the mode to allow simultaneous reading and writing
			'r+' read and write
			'w+' empty old content and write
			'a+' read and append
		append a 'U' to the mode open a file for input with universal new line support('\r', '\n', '\r\n')
			 'U' cannot be combined with 'w' or '+' mode.
	fp.close()

### read and write
> pydoc file

	fp.read()	Return file content.
	fp.readline()	Return just one line of text file.
	fp.write(string)	Writes string to file(start at file position)
	fp.writeline(string)	Writes string to file(start at file position)
	fp.truncate(...)
		truncate([size]) -> None.  Truncate the file to at most size bytes.
		Size defaults to the current file position, as returned by tell().

Example:

	fp = open('a.txt')
	print fp.read()
	fp.close();

### file position
`pydoc file`

	tell(...)
		tell() -> current file position, an integer (may be a long integer).
	seek(...)
		seek(offset[, whence]) -> None.  Move to new file position.
		Optional argument whence defaults to:
			0 (offset from start of file, offset should be >= 0); 
			1 (move relative to current position, positive or negative),
			2 (move relative to end of file, usually negative, although many platforms allow seeking beyond the end of a file).  
			If the file is opened in text mode, only offsets returned by tell() are legal.  Use of other offsets causes undefined behavior.

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

# function

	def func(s1,s2=None):
		global X
		local_var=4
		print global_var
		retrun s1,s2

	s1,s2 = func("-%s-" % 'Hello','-%s-' % 'Hilo' 'jack')
	print s1,s2

output:

	-Hello- -Hilojack-

## lambda function

	func = lambda x, y: x + y
	func = (lambda x, y: x + y)
	func(1,2)

lambda 不能显式使用return :

	# error
	func = lambda x, y: return x + y

## multi args

	def func(*args):
		s1,s2 = args

## named args

	func(a=1, b=2) equal func(b=2, a=1)

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

## loop

### for

	for i in range(5): print i
	for i in [1],2,3: print i

Iterator For

	[w.capitalize() for w in ['aa','bb','cc']]

### while

	while x<6:
		Statement

# logic

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

# import
wirte a `hilo.py`:

	def add(a, b):
		return a+b

Then under interact python envirment:
	
	import hilo
	print hilo.add(1,2)
	add = hilo.add
	print add(1,2)

	help(hilo) # What did you see?
	help(hilo.add)
	help(add)

import `ex47.game`, would find two file:

	ex47/game.py
	ex47/__init__.py

## from import

	from sys import argv # import module "sys" and objects: argv
	script, arg1, arg2 = argv

	import sys;
	sys.argv
