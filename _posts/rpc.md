---
layout: page
title:
category: blog
description:
---
# Preface
> http://www.cnblogs.com/fxjwind/archive/2013/05/16/3082219.html

rpc 问题其实是不同语言之间的数据通信问题:

1. 序列化问题, 怎么样将类对象或其他数据转化为用于传输的通用的格式, 如二进制, 文本, xml
2. 数据类型问题, 不同语言的数据类型的差异
3. 方法调用问题, 不同语言的方法调用的差异

当大数据时代来临的时候, 大家发现基于XML, 甚至Json的文本协议的方案的传输效率很成问题:
所以Google和Facebook, 又开始研究基于二进制的RPC方案, 于是产生PB, Thrift, Avro 等数据格式或协议

- protobuf(pb), binary
- thrift

数据格式

- xml XMPP就基于xml, 流量大
- msgpack, binary
- json, plain text

# PB(Protocol Buffer)
protobuf 是二进制数据格式协议，微信的短链接就是采用的protobuf, 与JSON 不同的是:
- binary
- 它自带了一个编译器，protoc，只需要用它进行编译，可以编译成JAVA、python、C++代码，暂时只有这三个

Refer to:
- http://www.searchtb.com/2012/09/protocol-buffers.html
- http://www.ibm.com/developerworks/cn/linux/l-cn-gpb/

# thrift
[thrift](/p/thrift)

# msgpack
msgpack 也是一个二进制的打包协议. 鸟哥的Yar http 框架默认作用该协议（也可以选择JSON)

# weixin 使用的协议
参考: http://blog.csdn.net/justinjing0612/article/details/38322353

为了保证稳定，微信用了长链接和短链接相结合，例如：

1 、两个域名

微信划分了http模式（short链接）和 tcp 模式（long 链接），分别应对状态协议和数据传输协议

long.weixin.qq.com  dns check （112.64.237.188 112.64.200.218）
 short.weixin.qq.com  dns check  ( 112.64.237.186 112.64.200.240)

2 说明

2.1 short.weixin.qq.com
是HTTP协议扩展，运行8080 端口，http body为二进制（protobuf）。

主要用途（接口）：

	用户登录验证;
	好友关系（获取，添加）；
	消息sync (newsync)，自有sync机制；
	获取用户图像；
	用户注销；
	行为日志上报。
	朋友圈发表刷新

 2.2  long.weixin.qq.com
tcp 长连接， 端口为8080，类似微软activesync的二进制协议。

主要用途（接口）：

	接受/发送文本消息；
	接受/发送语音；
	接受/发送图片；
	接受/发送视频文件等。

所有上面请求都是基于tcp长连接。在发送图片和视频文件等时，分为两个请求；第一个请求是缩略图的方式，第二个请求是全数据的方式。
