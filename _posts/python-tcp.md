---
layout: page
title:
category: blog
description:
---
# Preface

Socket是网络编程的一个抽象概念。通常我们用一个Socket表示“打开了一个网络链接”，而打开一个Socket需要知道目标计算机的IP地址和端口号，再指定协议类型即可。

# 客户端
大多数连接都是可靠的TCP连接。创建TCP连接时，主动发起连接的叫客户端，被动响应连接的叫服务器。

我们要创建一个基于TCP连接的Socket，可以这样做：

	import socket

	# 创建一个socket:
	s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
	# 建立连接:
	s.connect(('www.sina.com.cn', 80))

	# 发送数据:
	s.send(b'GET / HTTP/1.1\r\nHost: www.sina.com.cn\r\nConnection: close\r\n\r\n')

	# 接收数据:
	buffer = []
	while True:
		# 每次最多接收1k字节:
		d = s.recv(1024)
		if d:
			buffer.append(d)
		else:
			break
	data = b''.join(buffer)

	# 关闭连接:
	s.close()

	header, html = data.split(b'\r\n\r\n', 1)
	print(header.decode('utf-8'))
	# 把接收的数据写入文件:
	with open('sina.html', 'wb') as f:
	    f.write(html)

创建Socket时
1. AF_INET指定使用IPv4协议，如果要用更先进的IPv6，就指定为AF_INET6。
2. SOCK_STREAM指定使用面向流的TCP协议，这样，一个Socket对象就创建成功，但是还没有建立连接。

客户端要主动发起TCP连接，必须知道服务器的IP地址和端口号。新浪网站的IP地址可以用域名www.sina.com.cn自动转换到IP地址，但是怎么知道新浪服务器的端口号呢？

答案是作为服务器，提供什么样的服务，端口号就必须固定下来。例如:

1. SMTP服务是25端口，
2. FTP服务是21端口，等等。
3. 端口号小于1024的是Internet标准服务的端口，端口号大于1024的，可以任意使用。

# 服务器
和客户端编程相比，服务器编程就要复杂一些。

服务器进程首先要绑定一个端口并监听来自其他客户端的连接。如果某个客户端连接过来了，服务器就与该客户端建立Socket连接，随后的通信就靠这个Socket连接了。

所以，服务器会打开固定端口（比如80）监听，每来一个客户端连接，就创建该Socket连接。由于服务器会有大量来自客户端的连接，所以，服务器要能够区分一个Socket连接是和哪个客户端绑定的。一个Socket依赖4项：服务器地址、服务器端口、客户端地址、客户端端口来唯一确定一个Socket。

首先，创建一个基于IPv4和TCP协议的Socket：

	import socket, threading
	import time

	s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
	# 绑定、监听端口:
	s.bind(('127.0.0.1', 9999))
	s.listen(5); # 紧接着，调用listen()方法开始监听端口，传入的参数指定等待连接的最大数量：
	print('Waiting for connection...')

接下来，服务器程序通过一个永久循环来接受来自客户端的连接，accept()会等待并返回一个客户端的连接:

	while True:
		# 接受一个新连接:
		sock, addr = s.accept()
		# 创建新线程来处理TCP连接:
		t = threading.Thread(target=tcplink, args=(sock, addr))
		t.start()

每个连接都必须创建新线程（或进程）来处理，否则，单线程在处理连接的过程中，无法接受其他客户端的连接：

	def tcplink(sock, addr):
		print('Accept new connection from %s:%s...' % addr)
		sock.send(b'Welcome!')
		while True:
			data = sock.recv(1024)
			time.sleep(1)
			if not data or data.decode('utf-8') == 'exit':
				break
			sock.send(('Hello, %s!' % data.decode('utf-8')).encode('utf-8'))
		sock.close()
		print('Connection from %s:%s closed.' % addr)

连接建立后，服务器首先发一条欢迎消息，然后等待客户端数据，并加上Hello再发送给客户端。如果客户端发送了exit字符串，就直接关闭连接。

要测试这个服务器程序，我们还需要编写一个客户端程序：

	import socket, threading
	import time

	s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
	# 建立连接:
	s.connect(('127.0.0.1', 9999))
	# 接收欢迎消息:
	print(s.recv(1024).decode('utf-8'))
	for data in [b'Michael', b'Tracy', b'Sarah']:
	    # 发送数据:
	    s.send(data)
	    print(s.recv(1024).decode('utf-8'))
	    print(s.recv(1).decode('utf-8')); 它不是与send 对应的
	s.send(b'exit')
	s.close()

需要注意的是，客户端程序运行完毕就退出了，而服务器程序会永远运行下去，必须按Ctrl+C退出程序。
