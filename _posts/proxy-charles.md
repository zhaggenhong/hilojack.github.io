---
layout: page
title:	proxy 之charles
category: blog
description:
---
# Preface

# delete

	Cmd+del delete all session
	Shift+Cmd+P Mac OS proxy

# tools

	Shift+Cmd+w map rule
	Shift+Cmd+M map remote

# SSL

## 抓取https 数据
charles 需要在`Proxy`-`Proxy Settings - SSl` 中设置 `Enable SSL`, 且选择抓包的域名(`*` 代表所有)，charles 才会抓取https 数据。

否则，我们只能看到加密后的乱码数据。

![proxy-charles-1.png](/img/proxy-charles-1.png)

> 其实还有一步要做：客户端安装charles 证书。否则: 只能通过`curl -k -x '127.0.0.1:800' url` 跳过证书检查

## 给客户端装charles CA证书
http://www.charlesproxy.com/documentation/using-charles/ssl-certificates/

对于MAC OSX: "SSL Proxying > Install Charles Root Certificate".
IOS: 	browse to http://www.charlesproxy.com/getssl.
CHROME: "SSL Proxying > Save Charles Root Certificate"
JAVA:
	Choose "SSL Proxying > Save Charles Root Certificate".
	Find the cacerts file, it should be in your `$JAVA_HOME/jre/lib/security/cacerts`
	Then type :
		keytool -import -alias charles -file DESKTOP/charles-ssl-proxying-certificate.crt -keystore JAVA_HOME/jre/lib/security/cacerts -storepass changeit
		#(changeit is the default password on the cacerts file)
	Then try: keytool -list -keystore JAVA_HOME/jre/lib/security/cacerts -storepass changeit





