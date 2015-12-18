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

	list(range(start, end))

# Http
[python-http](/p/python-http)


# time

	import time
	print time.time()
		19972314124.05
	print int(time.time())

# string

## json

	import json
	return json.dumps(data)

# mysql

## MySQLdb

	import MySQLdb as mysql

	db = mysql.connect(user="reboot",passwd="reboot123",db="memory",host="localhost")
	db.autocommit(True)
	cur = db.cursor()

    sql = 'insert into memory (memory,time) value (%s,%s)'%(1024,1234)
    cur.execute(sql)
