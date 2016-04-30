---
layout: page
title:	shell proxy
category: blog
description:
---
# Preface

# In shell
Set http_proxy shell variable on Linux/OS X/Unix bash shell

Type the following command to set proxy server:

	$ export http_proxy=http://server-ip:port/
	export http_proxy=http://127.0.0.1:8888/

If the proxy server requires a username and password then add these to the URL. For example, to include the username foo and the password bar:

	$ export http_proxy=http://foo:bar@server-ip:port/
	$ export http_proxy=http://foo:bar@127.0.0.1:3128/
	$ export http_proxy=http://USERNAME:PASSWORD@proxy-server.mycorp.com:3128/

> 另外，还有一个https_proxy

## socks5
参考: [在终端下间接使用Socks5代理的几种方法(privoxy,tsocks,proxychains) ][privoxy,tsocks,proxychains]

### tsocks
需要这样使用 比如使用curl  的使用 tsocks curl 这样启动软件才能走代理

	sudo apt-get install tsocks
	vim /etc/tsocks.conf

brew install tsocks: https://github.com/Anakros/homebrew-tsocks

	tap repo: brew tap Anakros/homebrew-tsocks
	install tsocks: brew install --HEAD tsocks
	set up socks proxy: ssh -D 5555 server
	edit configuration file: vim /usr/local/etc/tsocks.conf
	set server as localhost: server = localhost
	set server port as 5555: server_port = 5555
	check that everything works: tsocks curl ifconfig.me

配置：

	#本地子网不使用代理
	local = 192.168.0.0/255.255.255.0
	local = 10.0.0.0/255.0.0.0
	local = 127.0.0.1/255.255.255.255

	#例外网站。这里列出来的网站不通过默认服务器走，而是通过特定的服务器走。server_type规定了这是一个socks5代理服务器
	path {
	        reaches = 150.0.0.0/255.255.0.0
	        reaches = 150.1.0.0:80/255.255.0.0
	        server = 10.1.7.25
	        server_type = 5
	        default_user = delius
	        default_pass = hello
	}

	# socks server
	server = 192.168.0.1
	# Server type defaults to 4 so we need to specify it as 5 for this one
	server_type = 5
	server_port = 1080

只是，tsocks 不支持`setuid program`(eg. [ssh](http://tsocks.sourceforge.net/faq.php#ssh)), 
> 其官方网站推荐了另外一个替代品,Dante，这个配置有点复杂

使用：

	$ tsocks wget ....

### proxychains with shadowsocks
proxychains 调用的好像是Privoxy

> https://github.com/shadowsocks/shadowsocks/wiki/Using-Shadowsocks-with-Command-Line-Tools

On Mac OS X:

	brew install proxychains-ng

Make a config file at `~/.proxychains/proxychains.conf` with content:

	strict_chain
	proxy_dns
	remote_dns_subnet 224
	tcp_read_time_out 15000
	tcp_connect_time_out 8000
	localnet 127.0.0.0/255.0.0.0
	quiet_mode

	[ProxyList]
	# type host port [user pass]
	socks5  127.0.0.1 1080

Then run command with proxychains. Examples:

	proxychains4 curl https://www.twitter.com/
	proxychains4 git push origin master
    proxychains4 curl 'http://1212.ip138.com/ic.asp'

Or just proxify bash:

	proxychains4 bash
	curl https://www.twitter.com/
	git push origin master

### Privoxy
Privoxy 能

1. 将http代理转发至 socks5/socks4 代理: 能支持http 代理的命令就可以用了
2. 支持类似PAC 的自动代理

	forward-socks5	/	127.0.0.1:1080	.    # socks v5
	forward-socks5t	/	127.0.0.1:1080	.    # socks v5, reduce the latency for the newly connection.

安装(Mac osx)

	$ brew install Privoxy
	echo "forward-socks5t   /               127.0.0.1:1080 .  " >> /usr/local/etc/privoxy/config
	gsed  -ir 's/^listen-address  127.0.0.1:8118/listen-address :8080/' /usr/local/etc/privoxy/config

启动

	$ brew info Privoxy
	To have launchd start privoxy at login:
	  ln -sfv /usr/local/opt/privoxy/*.plist ~/Library/LaunchAgents
	Then to load privoxy now:
	  launchctl load ~/Library/LaunchAgents/homebrew.mxcl.privoxy.plist
	Or, if you don't want/need launchctl, you can just run:
	  privoxy /usr/local/etc/privoxy/config

# iptables 全局代理
http://blog.csdn.net/decken_h/article/details/45306391
linux下实现全局  使用 iptables 做 REDIRECT

另外 Linux 里可以用 iptables + redsocks + shadowsocks 实现系统全局代理

> 试下 proxyCap

# global proxy
You can simply use wget command as follows:

	$ export https_proxy=http://proxy-server.mycorp.com:3128/
	$ export https_proxy=http://USERNAME:PASSWORD@proxy-server.mycorp.com:3128/
	$ wget --proxy-user=USERNAME --proxy-password=PASSWORD http://path.to.domain.com/some.html

Lynx command has the following syntax:

	$ lynx -pauth=USER:PASSWORD http://domain.com/path/html.file

Curl command has following syntax:

	$ curl --proxy-user user:password http://url.com/
	$ curl --proxy-user user:password -x proxy:port http://url.com/
	$ curl -U user:password -x proxy:port http://url.com/

	curl --socks5 127.0.0.1:1080

ab command

	-X proxy:port   Proxyserver and port number to use
	-P attribute    Add Basic Proxy Authentication, the attributes
                    are a colon separated username and password. "user:pwd"

# PAC Proxy
Proxies in network

## WPAD (Auto Proxy Discovery)
Setting up Web Proxy Autodiscovery Protocol (WPAD) using DNS
Refer to :
http://tektab.com/2012/09/26/setting-up-web-proxy-autodiscovery-protocol-wpad-using-dns/

1. To use WPAD using DNS method a DNS entry is needed for a host named WPAD. This name should be resolvable from the clients machine
1. Web server must be configured to serve the WPAD file with a MIME type of “application/x-ns-proxy-autoconfig”
1. A file named wpad.dat must be located in the WPAD Web server’s root directory.
1. The host at the WPAD address must be able to serve a Web page.
So if you are a member of example.com domain the browser is looking for this url for the PAC file http://wpad.example.com/wpad.dat

wpad.dat example:

	function FindProxyForURL(url, host) {
		var PROXY = "PROXY 192.168.0.100:8080";
		var PROXY_US = "SOCKS 192.168.0.100:9090";
		var PROXY_US_NEW = "PROXY 192.168.0.102:8080";
		var DEFAULT = "DIRECT";

		if(shExpMatch(host, "*pandora.com")) return PROXY_US;
		if(shExpMatch(host, "*blog.udn.com")) return PROXY;
		return DEFAULT;
	}

## Automatic Proxy Configuration
Proxy Auto Configuration file(PAC)

	http://127.0.0.1:8090/proxy.pac

1. In the `Network` Configure `Proxies` dropdown menu, select Using A PAC File
1. In the PAC File URL field, enter
`http://wpad/wpad.dat`
1. Click on Apply

# vpn
smart vpn
http://huzi.name/2015/04/28/mac-smart-vpn/

# Reference
- [在终端下间接使用Socks5代理的几种方法(privoxy,tsocks,proxychains) ][privoxy,tsocks,proxychains]

[privoxy,tsocks,proxychains]: http://blog.ihipop.info/2011/01/1988.html
