---
layout: page
title:
category: blog
description:
---
# Preface

# SQLite
SQLite是一种嵌入式数据库，它的数据库就是一个文件。由于SQLite本身是C写的，而且体积很小，所以，经常被集成到各种应用程序中，甚至在iOS和Android的App中都可以集成。

在使用SQLite前，我们先要搞清楚几个概念：

1. 表是数据库中存放关系数据的集合，一个数据库里面通常都包含多个表，比如学生的表，班级的表，学校的表，等等。表和表之间通过外键关联。
2. 要操作关系数据库，首先需要连接到数据库，一个数据库连接称为Connection；
3. 连接到数据库后，需要打开游标，称之为Cursor，通过Cursor执行SQL语句，然后，获得执行结果。

Python定义了一套操作数据库的API接口，任何数据库要连接到Python，只需要提供符合Python标准的数据库驱动即可: 
	由于SQLite的驱动内置在Python标准库中，所以我们可以直接来操作SQLite数据库。

我们在Python交互式命令行实践一下：

	# 导入SQLite驱动:
	>>> import sqlite3
	# 连接到SQLite数据库
	# 数据库文件是test.db
	# 如果文件不存在，会自动在当前目录创建:
	>>> conn = sqlite3.connect('test.db')
	# 创建一个Cursor:
	>>> cursor = conn.cursor()
	# 执行一条SQL语句，创建user表:
	>>> cursor.execute('create table user (id varchar(20) primary key, name varchar(20))')
	<sqlite3.Cursor object at 0x10f8aa260>
	# 继续执行一条SQL语句，插入一条记录:
	>>> cursor.execute('insert into user (id, name) values (\'1\', \'Michael\')')
	<sqlite3.Cursor object at 0x10f8aa260>
	# 通过rowcount获得插入的行数:
	>>> cursor.rowcount
	1
	# 关闭Cursor:
	>>> cursor.close()
	# 提交事务:
	>>> conn.commit()
	# 关闭Connection:
	>>> conn.close()

我们再试试查询记录：

	>>> conn = sqlite3.connect('test.db')
	>>> cursor = conn.cursor()
	# 执行查询语句:
	>>> cursor.execute('select * from user where id=?', '1')
	<sqlite3.Cursor object at 0x10f8aa340>
	# 获得查询结果集:
	>>> values = cursor.fetchall()
	>>> values
	[('1', 'Michael')]
	>>> cursor.close()
	>>> conn.close()

使用Python的DB-API时，只要搞清楚Connection和Cursor对象，打开后一定记得关闭，就可以放心地使用。
1. 使用Cursor对象执行`insert，update，delete`语句时，执行结果由`rowcount`返回影响的行数，就可以拿到执行结果。
2. 使用Cursor对象执行`select`语句时，通过`featchall()`可以拿到结果集。结果集是一个`list`，每个元素都是一个`tuple`，对应一行记录。

如果SQL语句带有参数，那么需要把参数按照位置传递给execute()方法，有几个?占位符就必须对应几个参数，例如：

	cursor.execute('select * from user where id=?', '1')

如何才能确保出错的情况下也关闭掉Connection对象和Cursor对象呢？请回忆try:...except:...finally:...的用法。

# Mysql
SQLite的特点是轻量级、可嵌入，但不能承受高并发访问，适合桌面和移动应用。而MySQL是为服务器端设计的数据库，能承受高并发访问，同时占用的内存也远远大于SQLite。

## 安装MySQL驱动
由于MySQL服务器以独立的进程运行，并通过网络对外服务，所以，需要支持Python的MySQL驱动来连接到MySQL服务器。MySQL官方提供了mysql-connector-python驱动，但是安装的时候需要给pip命令加上参数--allow-external：

	$ pip install mysql-connector-python --allow-external mysql-connector-python

## use mysql
我们演示如何连接到MySQL服务器的test数据库：

	# 导入MySQL驱动:
	>>> import mysql.connector
	# 注意把password设为你的root口令:
	>>> conn = mysql.connector.connect(user='root', password='password', database='test')
	>>> cursor = conn.cursor()
	# 创建user表:
	>>> cursor.execute('create table user (id varchar(20) primary key, name varchar(20))')
	# 插入一行记录，注意MySQL的占位符是%s:
	>>> cursor.execute('insert into user (id, name) values (%s, %s)', ['1', 'Michael'])
	>>> cursor.rowcount
	1
	# 提交事务:
	>>> conn.commit()
	>>> cursor.close()
	# 运行查询:
	>>> cursor = conn.cursor()
	>>> cursor.execute('select * from user where id = %s', ['1'])
	>>> values = cursor.fetchall()
	>>> values
	[('1', 'Michael')]
	# 关闭Cursor和Connection:
	>>> cursor.close()
	True
	>>> conn.close()

由于Python的DB-API定义都是通用的，所以，操作MySQL的数据库代码和SQLite类似。

# Grammar
如果要查k=>v这种格式的数据

	cursor = conn.cursor(dictionary=True)
