---
layout: page
title:
category: blog
description:
---
# Preface


# function

	def func(s1,s2=None):
		global X
		func.count++
		local_var=4
		print global_var
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

