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

## tsocks & proxychains
tsocks 或者 proxychains 让全局的网络到 privoxy, 它并非能像 windows下 Proxifier 这样全局网络代理.
而是需要这样使用 比如使用curl  的使用 tsocks curl 这样启动软件才能走代理

### proxychains with shadowsocks
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
	socks5  127.0.0.1 1080

Then run command with proxychains. Examples:

	proxychains4 curl https://www.twitter.com/
	proxychains4 git push origin master

Or just proxify bash:

	proxychains4 bash
	curl https://www.twitter.com/
	git push origin master

# iptables 全局代理
http://blog.csdn.net/decken_h/article/details/45306391
linux下实现全局  使用 iptables 做 REDIRECT

另外 Linux 里可以用 iptables + redsocks + shadowsocks 实现系统全局代理

> 试下 proxyCap

# Auth proxy
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
