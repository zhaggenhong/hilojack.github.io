---
layout: page
title: Shell 变量的数据类型
category: blog
description:
---
# Preface

shell 中的变量数据类型分为三种，

- 字符串
- 数字
- 数组

如果查看定义与查看一个变量是哪种类型呢？用`declare`:

	➜ > declare str=hello
	➜ > declare -p str
	typeset str=hello

	➜ > declare -i num=12
	➜ > declare -p num
	typeset -i num=12

	➜ > array=(1 2 3)
	➜ > declare -p array
	typeset -a array
	array=(1 2 3)

接下来本文将正式介绍字符串的使用

# 字符串的定义
如摘要所述，字符串定义可以使用`declare`, 或者`typeset`

	➜ > declare str=hello
	➜ > typset str=hello

其实是可以省略的：

	➜ > str=hello

> declare 与typeset 二者是等价命令


