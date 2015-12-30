---
layout: page
title:
category: blog
description:
---
# Preface

# regex


## ignore string Escape

	>>> print('abc\n001')
	abc
	001
	>>> print(r'abc\n001')
	abc\n001

## match
match()方法判断是否匹配，如果匹配成功，返回一个Match对象，否则返回None。常见的判断方法就是：

	>>> import re
	>>> m = re.match(r'^(\d{3})\-(\d{3,8})', '010-12345')
	<_sre.SRE_Match object; span=(0, 9), match='010-12345'>

### 分组:

	>>> m.group()
	010-12345
	>>> m.group(1)
	010
	>>> m.groups()
	('010', '12345')

### 去贪婪

	r'\d{3,8}?'
	r'\d+?'

## split
用正则表达式切分字符串比用固定的字符更灵活，请看正常的切分代码：

	>>> 'a b   c'.split(' ')
	['a', 'b', '', '', 'c']

嗯，无法识别连续的空格，用正则表达式试试：

	>>> re.split(r'[\s\,]+', 'a,b, c  d')
	['a', 'b', 'c', 'd']

## compile
果一个正则表达式要重复使用几千次，出于效率的考虑，我们可以预编译该正则表达式，接下来重复使用时就不需要编译这个步骤了，直接匹配：

	>>> re_telephone = re.compile(r'^(\d{3})-(\d{3,8})$')
	# 使用：
	>>> re_telephone.match('010-12345').groups()
	('010', '12345')
	>>> re_telephone.match('010-8086').groups()
	('010', '8086')



