---
layout: page
title:
category: blog
description:
---
# Preface

# 正则语法

## 分组:

	>>> import re
	>>> m = re.match(r'^(\d{3})\-(\d{3,8})', '010-12345')
	<_sre.SRE_Match object; span=(0, 9), match='010-12345'>

	>>> m.group()
	010-12345
	>>> m.group(1)
	010
	>>> m.groups()
	('010', '12345')

## 命名分组

	string = 'Hello foobar'
	pattern = re.search(r'(?P<fstar>f.*)(?P<bstar>b.*)', string)
	print "f* => {0}".format(pattern.group('fstar')) # prints f* => foo
	print "b* => {0}".format(pattern.group('bstar')) # prints b* => bar

## 去贪婪

	r'\d{3,8}?'
	r'\d+?'

## 条件(IF-Then-Else)模式
条件可以是一个数字。表示引用前面捕捉到的分组。

	(?(?=regex)then|else)

比如我们可以用这个正则表达式来检测打开和闭合的尖括号：

	strings = [  "<pypix>",    # returns true
				 "<foo",       # returns false
				 "bar>",       # returns false
				 "hello" ]     # returns true

	for string in strings:
		pattern = re.search(r'^(<)?[a-z]+(?(1)>)$', string)
		if pattern:
			print 'True'
		else:
			print 'False'

在上面的例子中，1 表示分组 (<)，当然也可以为空因为后面跟着一个问号。

# regex

	re.match(r'regex', str)
	re.compile(r'regex').match(str)

## compile
一个正则表达式要重复使用几千次，出于效率的考虑，我们可以预编译该正则表达式，接下来重复使用时就不需要编译这个步骤了，直接匹配：

	>>> re_telephone = re.compile(r'^(\d{3})-(\d{3,8})$')
	# 使用：
	>>> re_telephone.match('010-12345').groups()
	('010', '12345')
	>>> re_telephone.match('010-8086').groups()
	('010', '8086')

compile with modifier:

	re.compile(r'.*', re.S) # re.S == re.DOTALL

## ignore string Escape

	>>> print('abc\n001')
	abc
	001
	>>> print(r'abc\n001')
	abc\n001

## search

	>>> pattern = re.compile("d")
	>>> pattern.search(" dog")     # Match at index 1
	<_sre.SRE_Match object; span=(1, 2), match='d'>
	>>> pattern.search("dog", 2)  # No match; search doesn't include the "d"

### search info

	>>> re.search(r'abc', '1 abc').span()
	(2, 5)

If you want to locate a match anywhere in string, use search() .

## match fullmatch

	match() ^pattern
	fullmatch() ^pattern$

match()方法判断是否匹配，如果匹配成功，返回一个Match对象，否则返回None。


## split
用正则表达式切分字符串比用固定的字符更灵活，请看正常的切分代码：

	>>> 'a b   c'.split(' ')
	['a', 'b', '', '', 'c']

嗯，无法识别连续的空格，用正则表达式试试：

	>>> re.split(r'[\s\,]+', 'a,b, c  d')
	['a', 'b', 'c', 'd']

## findall
Use re.findall or re.finditer instead.

	re.findall(pattern, string) returns a list of matching strings.
	re.finditer(pattern, string) returns an iterator over MatchObject objects.

findall

	>>> re.findall(r'\w+', ' ahui jack')
	['ahui', 'jack']

## finditer

	>>> r=re.compile(r'(\w+)-(\w+)')
	>>> m=r.finditer(' 1-a1 2-a2')
	>>> for i in m:
	...     print(i.groups())
	...
	('1', 'a1')
	('2', 'a2')

## replace, sub
like `str.replace`

	re.sub(r'\w+', lambda m: m.group().upper(), ' hilo jack')
	' HILO JACK'

str.replace([list], rep) not work

