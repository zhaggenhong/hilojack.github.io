---
layout: page
title:
category: blog
description:
---
# Preface

# os

	>>> import os
	>>> os.name # 操作系统类型
	'posix'

要获取详细的系统信息，可以调用uname()函数：

	>>> os.uname()
	posix.uname_result(sysname='Darwin', nodename='MichaelMacPro.local', release='14.3.0', version='Darwin Kernel Version 14.3.0: Mon Mar 23 11:59:05 PDT 2015; root:xnu-2782.20.48~5/RELEASE_X86_64', machine='x86_64')

os cwd

	>>> import os
	>>> os.getcwd()
	'/home/user'
	>>> os.chdir("/tmp/")
	>>> os.getcwd()
	'/tmp'

# 环境变量
在操作系统中定义的环境变量，全部保存在os.environ这个变量中，可以直接查看：

	>>> os.environ
	environ({'VERSIONER_PYTHON_PREFER_32_BIT': 'no', 'TERM_PROGRAM_VERSION': '326', 'LOGNAME': 'michael', 'USER': 'michael', 'PATH': '/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/bin:/opt/X11/bin:/usr/local/mysql/bin', ...})
	要获取某个环境变量的值，可以调用os.environ.get('key')：

	>>> os.environ.get('PATH')
	'/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/bin:/opt/X11/bin:/usr/local/mysql/bin'
	>>> os.environ.get('x', 'default')
	'default'

# Directory

	from os.path import exists
	exists(file)

查看、创建和删除目录可以这么调用：

	# 查看当前目录的绝对路径:
	>>> os.path.abspath('.')
	'/Users/michael'
	# 在某个目录下创建一个新目录，首先把新目录的完整路径表示出来:
	>>> os.path.join('/Users/michael', 'testdir')
	'/Users/michael/testdir'
	# 然后创建一个目录:
	>>> os.mkdir('/Users/michael/testdir')
	# 删掉一个目录:
	>>> os.rmdir('/Users/michael/testdir')

## isfile, isdir

	>>> [x for x in os.listdir('.') if os.path.isfile(x) and os.path.splitext(x)[1]=='.py']
	['apis.py', 'config.py', 'models.py', 'pymonitor.py', 'test_db.py', 'urls.py', 'wsgiapp.py']

## pathinfo
拆分路径时，也不要直接去拆字符串，而要通过os.path.split()函数，这样可以把一个路径拆分为两部分，后一部分总是最后级别的目录或文件名：

	>>> os.path.split('/Users/michael/testdir/file.txt')
	('/Users/michael/testdir', 'file.txt')

	>>> os.path.splitext('/path/to/file.txt')
	('/path/to/file', '.txt')

# file

## 对文件重命名:

	>>> os.rename('test.txt', 'test.py')
	# 删掉文件:
	>>> os.remove('test.py')

## copyfile
shutil模块提供了copyfile()的函数，你还可以在shutil模块中找到很多实用函数，它们可以看做是os模块的补充。

