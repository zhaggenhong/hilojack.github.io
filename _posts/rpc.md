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
thrift 与protobuf 相比，不仅包括二进制数据格式部分，还包括网络部分: RPC、RMI、COM 等远程对象调用或远程方法调用。Protobuf 自身不带RPC 框架，需要第三方的RPC 框架支持(最近google 出了gRPC 框架)。

> thrift 比protobuf 重，文档也没有protobuf 丰富，我个人倾向于protobuf.

> google 最近开源了gRPC 框架，基于protobuf 数据打包协议和HTTP/2 协议. 提供了PHP/JAVA/PYTHON/GO 等多种实现

## install thrift

	brew install thrift
	git clone https://github.com/walkor/workerman-thrift

## 生成client
1. 写thrift

	thrift -gen php:server punish.thrift
	cat punish.thrift
		namespace php A.B.C

2. copy 组件

	cp -r ./gen-php/Services/HelloWorld /yourdir/workerman/applications/ThriftRpc/Services/

3. handler 组件
在`Applications/ThriftRpc/Services/HelloWorld/`目录下创建 `HelloWorldHandler.php`如下

	<?php
	// 统一使用Services\服务名 做为命名空间
	namespace Services\HelloWorld;

	class HelloWorldHandler implements HelloWorldIf {
	  public function sayHello($name)
	  {
		  return "Hello $name";
	  }
	}

4. 初始化

在Applications/ThriftRpc/start.php 中初始化服务，包括进端口和程数

	require_once __DIR__ . '/ThriftWorker.php';

	// helloworld
	$hello_worker = new ThriftWorker('tcp://0.0.0.0:9090');
	$hello_worker->count = 16;
	$hello_worker->class = 'HelloWorld';

	// another worker

5. client

	// 引入客户端文件
    require_once 'applications/ThriftRpc/Clients/ThriftClient.php';
    use ThriftClient\ThriftClient;

    // 传入配置，一般在某统一入口文件中调用一次该配置接口即可
    ThriftClient::config(array(
		 'HelloWorld' => array(
		   'addresses' => array(
			   '127.0.0.1:9090',
			   //'127.0.0.2:9191',
			 ),
			 'thrift_protocol' => 'TBinaryProtocol',//不配置默认是TBinaryProtocol，对应服务端HelloWorld.conf配置中的thrift_protocol
			 'thrift_transport' => 'TBufferedTransport',//不配置默认是TBufferedTransport，对应服务端HelloWorld.conf配置中的thrift_transport
		   ),
		   'UserInfo' => array(
			 'addresses' => array(
			   '127.0.0.1:9393'
			 ),
		   ),
		 )
	   );

    // 初始化一个HelloWorld的实例
    $client = ThriftClient::instance('HelloWorld');

    // --------同步调用实例----------
    var_export($client->sayHello("TOM"));

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
