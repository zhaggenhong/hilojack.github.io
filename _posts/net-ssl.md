---
layout: page
title:
category: blog
description:
---
# Preface
参考阮一峰的[ssl运行机制]，TLS/SSL 基于非对称加密，它解决了以下风险：

1. 窃听风险(eavesdropping): 通过私钥加密避免第三方窃取风险
2. 篡改风险(tampering): 通过检验, 在第三方修改数据时，通信双方可立即检测到. (伪造的私钥所加密的数据无法通过检测)
3. 冒充风险(pretending): 通过证书, 避免中间人攻击. (公钥是和证书一起的，证书可信，公钥就可信)

# todo

# TLS/SSL
传输层安全协议（Transport Layer Security，缩写为 TLS），
及其前身安全套接层（Secure Sockets Layer，SSL）是一种安全协议，目的是为互联网通信，提供安全及数据完整性保障。在网景公司（Netscape）推出首版Web浏览器的同时提出SSL，IETF将SSL进行标准化，1999年公布了 TLS标准文件。

TLS/SSL 本身属于应用层，高层的应用层协议（例如：HTTP、FTP、Telnet等等）能透明的建立于TLS协议之上。
它可和http 一起构成了https

TLS 有1.0 1.1 1.2 1.3 三个规范, 目前主流浏览器都支持 TLS1.2

## TLS 协议组成
> http://segmentfault.com/a/1190000002963044#articleHeader4
> https://segmentfault.com/a/1190000004631778#articleHeader3

协议由两层构成：`TLS Record Protocol`和`TLS Handshake Protocol`

`TLS Record Protocol`处于较低的一层，基于一些可信任的协议，如TCP，为高层协议提供数据封装、压缩、加密等基本功能的支持。它保证了通信的两个基本安全属性：

	1．保密连接。数据传输使用对称加密算法，如AES，RC4等，该对称加密算法的密钥对于每个连接是唯一的，基于密钥协商协议生成，比如TLS handshake protocol，Record Protocol也可以不使用加密。
	2．可信连接。消息的传输包括了基于密钥的消息认证码（keyed MAC），使用安全Hash函数计算MAC，用于完整性检查。Record Protocol也可以不使用MAC，但是这种模式只用于安全参数协商时。

`Record Protocol`用于封装多种高层的协议，其中一个种就是`TLS handshake protocol`，这种协议允许客户与服务器相互认证，在应用程序通信前，协商加密算法和加密密钥。`TLS handshake protocol`保证了连接的三个基本安全属性：

	1．两端的身份可以通过非对称或者公钥加密算法（DSA，RSA等）进行认证。认证过程是可选的，但至少要求一端被认证。
	2．共享密钥的协商是安全的。密钥协商对于监听者和任何被认证的连接都是不可见的。
	3．协商是可信的。攻击者无法修改协商信息。

TLS协议提供的服务主要有：

	1）认证用户和服务器，确保数据发送到正确的客户机和服务器；
	2）加密数据以防止数据中途被攻击者监听；
	3）维护数据的完整性，确保数据在传输过程中不被攻击者篡改。

TLS协议在协议栈中如下图所示：


## 通信过程
SSL/TLS 是基于非对称加密的，客户端需要通过服务端的公钥以确认服务端的身份，所以：

*确认服务器的身份*

1. 客户端需要先拿到服务端的公钥, 这个公钥放在服务端的证书中。但是如何确认证书可信？
2. CA 确认

*生成对话密钥*
确认身份后，如何通信呢? 如果用服务端的公钥/私钥通信，会存在一些问题：

1. 如果用客户端用服务端的公钥加密信息A，第三方也可以用这个公钥加密信息, 但是第三方拿不到原始的消息A. 所以第三方无法冒充客户发送消息
2. 如果服务用私钥加密消息，客户端和第三方都有公钥，大家都可以解出这个消息. 所以服务器的私钥加密的消息必须是公开性质的，比如服务器的证书.
	所以服务端返回的消息需要用其它密钥加密，比如会话密钥。
3. 非对称加密速度非常慢, 而对称密钥通信很快。

所以实际的SSL 数据通信方法是：
每次会话(session) 客户端和服务端之间都生成一个对话密钥(session key), 这个基于对称加密的密钥 用于加密数据，速度非常快

*进行加密通信*

总结一下通信过程：

	（1） 客户端向服务器端索要并验证公钥。
	（2） 双方协商生成"对话密钥"。
	（3） 双方采用"对话密钥"进行加密通信。

## 密钥交换和密钥协商
在客户端和服务器开始交换 TLS 所保护的加密信息之前，他们必须安全地交换或协定加密密钥和加密数据时要使用的密码。

用于密钥交换的方法包括：

- 使用 RSA 算法生成公钥和私钥（在 TLS 握手协议中被称为 TLS_RSA），
- Diffie-Hellman（在 TLS 握手协议中被称为 TLS_DH），
- 临时 Diffie-Hellman（在 TLS 握手协议中被称为 TLS_DHE），
- 椭圆曲线 Diffie-Hellman （在 TLS 握手协议中被称为 TLS_ECDH），
- 临时椭圆曲线 Diffie-Hellman（在 TLS 握手协议中被称为 TLS_ECDHE），
- 匿名 Diffie-Hellman（在 TLS 握手协议中被称为 TLS_DH_anon）[12]
- 和预共享密钥（在 TLS 握手协议中被称为 TLS_PSK）。[13]

只有临时 TLS_DHE 和 TLS_ECDHE 提供前向保密能力。

# generate self certificate key
Refer to: http://www.lovelucy.info/nginx-ssl-certificate-https-website.html

通常使用开源的openssl 生成ssl 证书。
OpenSSL支持多种不同的加密算法, 它有LibreSSL 和 google 的BoringSSL 两个分支

- 加密：
AES, Blowfish, Camellia, SEED, CAST-128, DES, IDEA, RC2, RC4, RC5, Triple DES, GOST 28147-89[3]

- 散列函数：
	MD5, MD2, SHA-1, SHA-2, RIPEMD-160, MDC-2, GOST R 34.11-94[3]

- 公开密钥加密：
	RSA, DSA, Diffie–Hellman key exchange, Elliptic curve, GOST R 34.10-2001[3]

`openssl` 可以用来生成ssl 密钥(.key), 证书请求(.csr), 签名证书(.crt).

下例生成一个自签名证书(self sign certificate), 而非[ssl-ca](/p/ssl-ca) 中CA 签名证书(`CA -sign`)

	# 生成一个RSA密钥
	openssl genrsa -des3 -out my.key 1024

	# 拷贝一个不需要输入密码的密钥文件(public)
	openssl rsa -in my.key -out my_nopass.key

	# 生成一个证书请求
	openssl req -new -key my.key -out my.csr
	#会提示输入省份、城市、域名信息等 //common name 一定要填写实际的域名，否则浏览器会报: ERR_CERT_COMMON_NAME_INVAID

	# 自己签发证书
	openssl x509 -req -days 365 -in my.csr -signkey my.key -out my.crt

这样就有一个 csr 文件了，提交给 ssl 提供商的时候就是这个 csr 文件。当然我这里并没有向证书提供商申请，而是在第4步自己签发了证书。

对了`common name = *.weibo.cn`, chrome 匹配`a.wiki.cn`, 但是不会认匹配`a.b.wiki.cn`, safari 则都匹配

For more details:

- http://datacenteroverlords.com/2012/03/01/creating-your-own-ssl-certificate-authority/

> 根证书(root.pem)本质上是自签名证书，根证书可以用于对其它子证书签名(参考上面的链接)

# nginx

	server {
		listen       80;
		listen 443 default_server ssl;
		ssl_certificate /Users/hilojack/test/ssl/my.crt;
		ssl_certificate_key /Users/hilojack/test/ssl/my_nopass.key;

		# 若ssl_certificate_key使用my.key，则每次启动Nginx服务器都要求输入key的密码。
		#ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;#defalut
		#ssl_ciphers         HIGH:!aNULL:!MD5; #default
	}

# Import Certificate

## On MacOSX
1. Open Keychain
2. `File->Import Items(Shift+Cmd+i)` to import `my.crt`, `ca.crt` in `system`
3. Select *always trust*

# debug

## Certificate chain

	$ openssl s_client -connect www.godaddy.com:443
	...
	Certificate chain
	 0 s:/C=US/ST=Arizona/L=Scottsdale/1.3.6.1.4.1.311.60.2.1.3=US
		 /1.3.6.1.4.1.311.60.2.1.2=AZ/O=GoDaddy.com, Inc
		 /OU=MIS Department/CN=www.GoDaddy.com
		 /serialNumber=0796928-7/2.5.4.15=V1.0, Clause 5.(b)
	   i:/C=US/ST=Arizona/L=Scottsdale/O=GoDaddy.com, Inc.
		 /OU=http://certificates.godaddy.com/repository
		 /CN=Go Daddy Secure Certification Authority
		 /serialNumber=07969287
	 1 s:/C=US/ST=Arizona/L=Scottsdale/O=GoDaddy.com, Inc.
		 /OU=http://certificates.godaddy.com/repository
		 /CN=Go Daddy Secure Certification Authority
		 /serialNumber=07969287
	   i:/C=US/O=The Go Daddy Group, Inc.
		 /OU=Go Daddy Class 2 Certification Authority
	 2 s:/C=US/O=The Go Daddy Group, Inc.
		 /OU=Go Daddy Class 2 Certification Authority
	   i:/L=ValiCert Validation Network/O=ValiCert, Inc.
		 /OU=ValiCert Class 2 Policy Validation Authority
		 /CN=http://www.valicert.com//emailAddress=info@valicert.com

# Reference
- [nginx_https]
- [wiki_ssl]
- [ssl运行机制]
- [illustration-ssl]
- [nginx ssl]

[nginx ssl]: http://www.lovelucy.info/nginx-ssl-certificate-https-website.html
[ssl运行机制]: http://www.ruanyifeng.com/blog/2014/02/ssl_tls.html
[illustration-ssl]: http://www.ruanyifeng.com/blog/2014/09/illustration-ssl.html
[wiki_ssl]: http://zh.wikipedia.org/wiki/%E5%82%B3%E8%BC%B8%E5%B1%A4%E5%AE%89%E5%85%A8%E5%8D%94%E8%AD%B0
[nginx_https]: http://nginx.org/cn/docs/http/configuring_https_servers.html
