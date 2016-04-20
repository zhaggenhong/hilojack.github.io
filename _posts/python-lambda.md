---
layout: page
title:
category: blog
description:
---
# Preface

# lambda 与 惰性求值
> https://segmentfault.com/a/1190000004955195

python对函数式编程有一定的支持，具体表现在高级函数,map,reduce,filter,partial function等几个方面。在应用函数式编程中，需要关注函数式编程中的变量不变性，惰性计算等原则，否则容易踩到坑。

	def f():
	   def num_gen():
		   x = 2
		   while True:
			   yield x
			   x+=1
	   l=num_gen()
	   while True:
		   n = next(l)
		   yield n
		   l = filter(lambda x:x%n!=0,l)
	primes = f()
	for i in primes:
		print(i)

这段代码本意在于返回一个素数列表，但实际在运行中返回的并不是想要的。究其原因，在于filter的惰性计算以及lambda匿名函数的闭包特性。上面的代码中，n是一个变量，lambda匿名函数只有在next(f())时才会真正调用，而此时，n的值已经是当前n的值，而不是之前的值了，比如9，应当是3的倍数，但此时n的值并不是3，导致9%n>0成立。

要修改这段代码，一个方法是利用lambda的闭包，避开n变量，生成一个和n无关的函数。

	def f():
	   def g(n):
		   return lambda x:x%n!=0
	   def num_gen():
		   x = 2
		   while True:
			   yield x
			   x+=1
	   l = num_gen()
	   while True:
		   n = next(l)
		   yield n
		   l = filter(g(n),l)
	primes = f()
	for i in primes:
		print(i)

这样修改后，g(n)是实时调用的，返回一个和n无关的函数，这样在next惰性调用时，就不会出错。

## 总结

	def r(m,n):
		print(m)
		return n
	n=1
	f=lambda x:r(n,x); # n=1???
	o = filter(f,range(1,5));
	n=100
	for i in o: print('=======')


类似的错误在返回函数的高阶函数中也容易出错，如果返回的函数和一个变量绑定了，就会出现问题，解决的方法都是一样，增加一个闭包函数，使得返回函数和变量无关。
