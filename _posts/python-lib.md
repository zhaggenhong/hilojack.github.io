---
layout: page
title:	
category: blog
description: 
---
# Preface


# Test
assert False, "Error!"

# Yield

	def X(): yield Y; X().next()

# encrypt

## hash

	hash('a');//12416037344

## base64

	>>> import base64
	>>> import pickle
	>>> base64.b64decode(open('a.txt').read())
	"(dp1\nS'count'\np2\nI5\nsS'ip'\np3\nV127.0.0.1\np4\nsS'session_id'\np5\nS'df7cd559362a9d170ecb1a4e7752c5ab999fb0eb'\np6\ns."
	>>> x =base64.b64decode(open('a.txt').read())
	>>> pickle.loads(x)
	{'count': 5, 'ip': u'127.0.0.1', 'session_id': 'df7cd559362a9d170ecb1a4e7752c5ab999fb0eb'}

# Math

	import random
	i=random.randint(1, 20)

## range

	list(range([start=0,] end))

# Http
[python-http](/p/python-http)

# time

	import time
	print time.time()
		19972314124.05
	print int(time.time())

# string

## hex

	hex(16)

## url

	import urllib
	urllib.parse.parse_qs('a=1&b=2');
	urllib.parse.parse_qs('a=1&b=2',unique_key=True);

	urllib.parse.urlencode({'a':1})
	'a=1'
	urllib.parse.urlencode({'a':[1,2]}, doseq=True)
	'a=1&a=2'

## json

	import json
	str = json.dumps(data)
	print(json.loads(str))

### json class
json dumps 类时，得用default

	class Student(object):
		def __init__(self, name, age, score):
			self.name = name
			self.age = age
			self.score = score

	s = Student('Bob', 20, 88)
	>>> print(json.dumps(s, default=lambda m: return { 'name': std.name, 'age': std.age, 'score': std.score }))
	{"age": 20, "name": "Bob", "score": 88}

反序列化为class

	def dict2student(d):
		return Student(d['name'], d['age'], d['score'])
	运行结果如下：

	>>> json_str = '{"age": 20, "score": 88, "name": "Bob"}'
	>>> print(json.loads(json_str, object_hook=dict2student))

# mysql

## MySQLdb

	import MySQLdb as mysql

	db = mysql.connect(user="reboot",passwd="reboot123",db="memory",host="localhost")
	db.autocommit(True)
	cur = db.cursor()

    sql = 'insert into memory (memory,time) value (%s,%s)'%(1024,1234)
    cur.execute(sql)
