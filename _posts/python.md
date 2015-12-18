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

## in python

	> help(file)
	var=1
	> help(var)

# Shell 

	cat a.py | python
	cat a.py | python - arg1

Note: raw_input 将不会正常工作，We better use :

	python <(cat p.py)
	python p.py
	

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

## path

	>>> import sys
	>>> print(sys.path)
	['', '/usr/local/Cellar/python3/3.5.0/Frameworks/Python.framework/Versions/3.5/lib/python35.zip', '/usr/local/Cellar/python3/3.5.0/Frameworks/Python.framework/Versions/3.5/lib/python3.5', '/usr/local/Cellar/python3/3.5.0/Frameworks/Python.framework/Versions/3.5/lib/python3.5/plat-darwin', '/usr/local/Cellar/python3/3.5.0/Frameworks/Python.framework/Versions/3.5/lib/python3.5/lib-dynload', '/usr/local/lib/python3.5/site-packages']

## from import

	from sys import argv # import module "sys" and objects: argv
	script, arg1, arg2 = argv

	import sys;
	sys.argv
