---
layout: page
title:
category: blog
description:
---
# Preface

# Request
http://docs.python-requests.org/en/latest/index.html

## Non Block
If you are concerned about the use of blocking IO, there are lots of projects out there that combine Requests with one of Python’s asynchronicity frameworks. Two excellent examples are grequests and requests-futures.

## install

	easy_install requests //2.7
	pip3 install requests
	import requests

## request

	>>> r = requests.get('https://127.0.0.1:8000/a.php')
	>>> r = requests.post("http://httpbin.org/post", data = {"key":"value"})
	>>> r = requests.put("http://httpbin.org/put", data = {"key":"value"})
	>>> r = requests.delete("http://httpbin.org/delete")
	>>> r = requests.head("http://httpbin.org/get")
	>>> r = requests.options("http://httpbin.org/get")

### params in url
Passing Parameters In URLs

	>>> payload = {'key1': 'value1', 'key2': 'value2'}
	>>> r = requests.get("http://httpbin.org/get", params=payload)
	>>> print(r.url)
		http://httpbin.org/get?key2=value2&key1=value1

### post json

	>>> import json
	>>> payload = {'some': 'data'}
	>>> r = requests.post(url, data=json.dumps(payload))

Instead of encoding the dict yourself, you can also pass it directly using the json parameter (added in version 2.4.2) and it will be encoded automatically:

	>>> payload = {'some': 'data'}
	>>> r = requests.post(url, json=payload)

### post a Multipart-Encoded File

	>>> files = {'file': open('report.xls', 'rb')}
	>>> r = requests.post(url, files=files)

You can set the filename, content_type and headers explicitly:

	>>> files = {'file': ('report.xls', open('report.xls', 'rb'), 'application/vnd.ms-excel', {'Expires': '0'})}
	>>> r = requests.post(url, files=files)

If you want, you can send strings to be received as files:

	>>> files = {'file': ('report.csv', 'some,data,to,send\nanother,row,to,send\n')}
	>>> r = requests.post(url, files=files)

## Custom Headers
For example, we didn’t specify our user-agent in the previous example:

	>>> url = 'https://api.github.com/some/endpoint'
	>>> headers = {'user-agent': 'my-app/0.0.1'}

	>>> r = requests.get(url, headers=headers)


## Response Content

### response text

	>>> r.text
	u'[{"repository":{"open_issues":0,"url":"https://github.com/...
	>>> r.encoding
	'utf-8'
	>>> r.encoding = 'ISO-8859-1'

### response binary content("bytes")

	>>> r.content
	b'[{"repository":{"open_issues":0,"url":"https://github.com/...

	 //to create an image from binary data returned by a request, you can use the following code:
	>>> from PIL import Image
	>>> from StringIO import StringIO
	>>> i = Image.open(StringIO(r.content))

### Response json

	r.json()['key']

### Raw Response Content

In the rare case that you’d like to get the raw socket response from the server, you can access r.raw. If you want to do this, make sure you set stream=True in your initial request. Once you do, you can do this:

	>>> r = requests.get('https://api.github.com/events', stream=True)
	>>> r.raw
	<requests.packages.urllib3.response.HTTPResponse object at 0x101194810>
	>>> r.raw.read(10)
	'\x1f\x8b\x08\x00\x00\x00\x00\x00\x00\x03'

In general, however, you should use a pattern like this to save what is being streamed to a file:

	with open(filename, 'wb') as fd:
		for chunk in r.iter_content(chunk_size):
			fd.write(chunk)

### response status_code

	>>> r.status_code
	200

	>>> r.status_code == requests.codes.ok
	True

### Response Headers
We can view the server’s response headers using a Python dictionary:

	>>> r.headers
	{
		'content-encoding': 'gzip',
		'transfer-encoding': 'chunked',
		'connection': 'close',
		'content-type': 'application/json'
	}

The dictionary is special, though: it’s made just for HTTP headers. According to RFC 7230, HTTP Header names are case-insensitive.

	>>> r.headers['Content-Type']
	'application/json'

	>>> r.headers.get('content-type')
	'application/json'

### Cookie
If a response contains some Cookies, you can quickly access them:

	>>> r.cookies['example_cookie_name']
	'example_cookie_value'

To send your own cookies to the server, you can use the cookies parameter:

	>>> r = requests.get(url, cookies={'key':'value'})
	>>> r = requests.get(url, cookies='key=value')

	>>> r1 = requests.post('http://www.yourapp.com/login')
	>>> r2 = requests.post('http://www.yourapp.com/somepage',cookies=r1.cookies)

keep cookie

	>>> s.get('https://www.google.com', cookies = {'cookieKey':'cookieValue'})

### Redirection and History
By default Requests will perform location redirection for all verbs except HEAD.

The Response.history list contains the Response objects that were created in order to complete the request. The list is sorted from the oldest to the most recent response.

	>>> r = requests.get('http://github.com')
	>>> r.url
	'https://github.com/'
	>>> r.status_code
	200
	>>> r.history
	[<Response [301]>]

If you’re using GET, OPTIONS, POST, PUT, PATCH or DELETE, you can disable redirection handling with the allow_redirects parameter:

	>>> r = requests.get('http://github.com', allow_redirects=False)
	>>> r.status_code
	301
	>>> r.history
	[]

If you’re using HEAD, you can enable redirection as well:

	>>> r = requests.head('http://github.com', allow_redirects=True)

## Timeouts
You can tell Requests to stop waiting for a response after a given number of seconds with the timeout parameter:

	>>> requests.get('http://github.com', timeout=0.001)
	Traceback (most recent call last):
	  File "<stdin>", line 1, in <module>
	requests.exceptions.Timeout: HTTPConnectionPool(host='github.com', port=80): Request timed out. (timeout=0.001)

## Session
Use a session object instead, it'll persist cookies and send them back to the server:

	with requests.Session() as s:
		r = s.get(URL1)
		r = s.post(URL2, params="username and password data payload")

取设headers:

	s = requests.Session()
	s.auth = ('user', 'pass')
	s.headers.update({'x-test': 'true'})
	# 如何构造？？help(*cookie_jar)
	s.cookies.set_cookie('a=1;b=2;domain/')? tetst

	// both 'x-test' and 'x-test2' are sent
	s.get('http://httpbin.org/headers', headers={'x-test2': 'true'})

# urllib2
urllib2 is deprecated

## proxy

	import urllib
	import urllib2
	url = 'http://weibo.cn'
	data = urllib.urlencode({'k':'v'})
	opener = urllib2.build_opener(urllib2.ProxyHandler({'http':'http://ip:port'}))
	urllib2.install_opener(opener)
	req = urllib2.Request(url, data)
	try:
		response = urllib2.urlopen(req,timeout=2)
		the_page = response.read()
		print the_page.decode('utf-8')
	except:
		pass
