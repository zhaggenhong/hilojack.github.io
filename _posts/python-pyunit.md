---
layout: page
title:
category: blog
description:
---
# Preface
> 参考：http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/00143191629979802b566644aa84656b50cd484ec4a7838000

编写一个Dict类，这个类的行为和dict一致，但是可以通过属性来访问，用起来就像下面这样：

	>>> d = Dict(a=1, b=2)
	>>> d['a']
	1
	>>> d.a
	1

	class Dict(dict):

		def __init__(self, **kw):
			super().__init__(**kw)

		def __getattr__(self, key):
			try:
				return self[key]
			except KeyError:
				raise AttributeError(r"'Dict' object has no attribute '%s'" % key)

		def __setattr__(self, key, value):
			self[key] = value

# 测试类
编写一个测试类，从unittest.TestCase继承

	import unittest
	from mydict import Dict

	class TestDict(unittest.TestCase):

		def test_init(self):
			d = Dict(a=1, b='test')
			self.assertEqual(d.a, 1)
			self.assertEqual(d.b, 'test')
			self.assertTrue(isinstance(d, dict))

		def test_key(self):
			d = Dict()
			d['key'] = 'value'
			self.assertEqual(d.key, 'value')

		def test_attr(self):
			d = Dict()
			d.key = 'value'
			self.assertTrue('key' in d)
			self.assertEqual(d['key'], 'value')

		def test_keyerror(self):
			d = Dict()
			with self.assertRaises(KeyError):
				value = d['empty']

		def test_attrerror(self):
			d = Dict()
			with self.assertRaises(AttributeError):
				value = d.empty

以test开头的方法就是测试方法，不以test开头的方法不被认为是测试方法，测试的时候不会被执行。

## 断言

	self.assertEqual(abs(-1), 1) # 断言函数返回的结果与1相等

另一种重要的断言就是期待抛出指定类型的Error，比如通过d['empty']访问不存在的key时，断言会抛出KeyError：

	with self.assertRaises(KeyError):
		value = d['empty']
