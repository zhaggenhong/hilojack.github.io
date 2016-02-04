---
layout: page
title:	Learn Python
category: blog
description:
---
# Preface

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
http://www.diveintopython3.net/
http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000

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

## exec python

	cat a.py | python
	cat a.py | python - arg1

Note: raw_input 将不会正常工作，We better use :

	python <(cat p.py)
	python p.py

## exec shell

### subprocess.Popen
Popen 是最基础的类, 它是非阻塞的(除非有`.stdout.read()`)

	subprocess.Popen('sleep 61') #wrong
	subprocess.Popen(['sleep', '61']).pid; #ok

	subprocess.Popen(['echo','12'], stdout=subprocess.PIPE, stderr=subprocess.PIPE, stdin=subprocess.PIPE)

read output

	proc = subprocess.Popen(['ls','-l'],stdout=subprocess.PIPE)
	line = proc.stdout.readline();
	while line;
		print "test:", line.rstrip()
		line = proc.stdout.readline();

#### shell=True 时:
1. 只取第一个参数，且把参数当成shell 语句而非命令
2. 可以执行multiple commands

	subprocess.Popen('sleep 5;echo abc',shell=True, stdout=subprocess.PIPE).stdout.read()

### via call
类似run, 但

1. 只返回errno

	>>from subprocess import call
	>>errno=call("ls -l", shell=True)
	total 240
	-rw-r--r--   1 hilojack  staff   1.6K Dec 16 13:26 a.txt

### via run
它封装(package)的`subprocess.Popen`

	>>> import subprocess

doesn't capture output

	>>> subprocess.run(["ls", "-l", "/dev/null"])
	crw-rw-rw-  1 root  wheel    3,   2 Dec 19 11:13 /dev/null
	CompletedProcess(args=['ls', '-l', '/dev/null'], returncode=0)

capture output

	>>> subprocess.run(["ls", "-l", "/dev/null"], stdout=subprocess.PIPE)
	CompletedProcess(args=['ls', '-l', '/dev/null'], returncode=0, stdout=b'crw-rw-rw-  1 root  wheel    3,   2 Dec 19 11:13 /dev/null\n')
	>>> subprocess.run(["ls", "-l", "/dev/null"], stdout=subprocess.PIPE).stdout
	crw-rw-rw-  1 root  wheel    3,   2 Dec 19 11:13 /dev/null

check error

	>>> subprocess.run("exit 1", shell=True)
	CompletedProcess(args='exit 1', returncode=1)

	>>> subprocess.run("exit 1", shell=True, check=True)
	Traceback (most recent call last):
	  File "<stdin>", line 1, in <module>
	  File "/usr/local/Cellar/python3/3.5.0/Frameworks/Python.framework/Versions/3.5/lib/python3.5/subprocess.py", line 711, in run
		output=stdout, stderr=stderr)
	subprocess.CalledProcessError: Command 'exit 1' returned non-zero exit status 1

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

请注意，每一个包目录下面都会有一个__init__.py的文件，这个文件是必须存在的，否则，Python就把这个目录当成普通目录，而不是一个包。__init__.py可以是空文件，也可以有Python代码，因为__init__.py本身就是一个模块，而它的模块名就是mycompany。

## __name__
命令行执行模块时，`__name__` = `__main__` , import 导入时，它等于模块名

	if __name__=='__main__':
		print(__name__)

## scope
模块内变量作用scope

1. public:
	正常的函数和变量名是公开的（public），可以被直接引用，比如：abc，x123，PI等；

1. 特殊变量`__xxx__`： 可以被直接引用，但是有特殊用途
	__author__，__name__就是特殊变量
	__doc__ 文档注释也可以用特殊变量

3. `_xxx和__xxx`
	这样的函数或变量就是非公开的（private），不应该被直接引用，比如_abc，__abc等；

> 之所以我们说，private函数和变量“不应该”被直接引用，而不是“不能”被直接引用，是因为Python并没有一种方法可以完全限制访问private函数或变量

## path
此PATH 与 SHELL PATH 是独立的

### sys.path

	>>> import sys
	>>> sys.path.append('path')
	>>> print(sys.path)
	['', '/usr/local/Cellar/python3/3.5.0/Frameworks/Python.framework/Versions/3.5/lib/python35.zip', '/usr/local/Cellar/python3/3.5.0/Frameworks/Python.framework/Versions/3.5/lib/python3.5', '/usr/local/Cellar/python3/3.5.0/Frameworks/Python.framework/Versions/3.5/lib/python3.5/plat-darwin', '/usr/local/Cellar/python3/3.5.0/Frameworks/Python.framework/Versions/3.5/lib/python3.5/lib-dynload', '/usr/local/lib/python3.5/site-packages']
	 '/usr/local/lib/python3.5/site-packages'
	 /usr/local/lib/python3.5/site-packages/pip/_vendor/requests/cookies.py

### PYTHONPATH
相当于sys.path.append()

	export PYTHONPATH=.

## from import

	from sys import argv # import module "sys" and objects: argv
	script, arg1, arg2 = argv

	import sys;
	sys.argv

# install

## pip3

	easy_install requests //2.7
	pip3 install requests
	import requests

