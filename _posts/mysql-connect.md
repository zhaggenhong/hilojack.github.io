---
layout: page
title:	
category: blog
description: 
---
# Preface


# mysql 长连接

永久连接一样是每个客户端来就打开一个连接，有200人访问就有200个连接。

其 实mysql_pconnect()本身并没有做太多的处理, 它唯一做的只是在php运行结束后不主动close掉mysql的连接. 

1. 在php经cgi方式运行时pconnect和connect是基本没有区别的, 因为cgi方式是每一个php访问起一个进程, 访问结束后进程也就结束了, 资源也全释放了.

2. 当php以apache模块方式运行时, 由于apache有使用进程池, 一个httpd进程结束后会被放回进程池, 这也就使得用pconnect打开的的那个mysql连接资源不被释放, 于是有下一个连接请求时就可以被复用.

这就使得在apache并发访问量大的时候, 由于使用了pconnect, php节省了反复连接db的时间, 使得访问速度加快. 这应该是比较好理解的. 

