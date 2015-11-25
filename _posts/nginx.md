---
layout: page
title:	linux nginx 配置
category: blog
description:
---
# Preface
nginx是比apache 更先进的web server. 以BSD许可证发布. 内核简洁, 模块强大(与apache不同, 其所有的模块都是静态编译的, 更快). 简单的是nginx接收到http请求后. 当分析到请求是js/css/img等静态资源, 就交给静态资源模块去处理. 如果是php/python等动态资源, 就交给FastCGI去处理.

> FastCGI = web server 和 动态脚本语言的接口

## I/O消息通知机制
Nginx支持epoll(linux系) 和 qkqueue(bsd) 系两种事件通知机制. epoll是linux2.6引入的提高I/O的处理方法, 优点: 单一进程打开的文件描述符(fd)仅受操作系统限制; 采用内存共享以避免内存copy; fd打开的数量的增加, 不会使I/O性能线性下降.


## 多进程
niginx启动后会有一个master进程(负责接收外界信号并向worker发送信号, 监控worker) 和多个worker进程(一般对应cpu数量)

nginx 每进来一个request，会有一个worker进程去处理。但不是全程的处理，处理到什么程度呢？处理到可能发生阻塞的地方，比如向上游（后端）服务器转发request，并等待请求返回。那么，这个处理的worker不会这么傻等着，他会在发送完请求后，注册一个事件：“如果upstream返回了，告诉我一声，我再接着干”。于是他就休息去了。此时，如果再有request 进来，他就可以很快再按这种方式处理。而一旦上游服务器返回了，就会触发这个事件，worker才会来接手，这个request才会接着往下走。 @金由

如果master发现, worker死掉了, 它会再开一个worker. worker会用到epoll通知机制.

	worker_rlimit_nofile 51200; #文件描述符数量限制
	worker_processes 4 #如果有4个cpu
	events {
		# use [ kqueue | rtsig | epoll | /dev/poll | select | poll ];
		use epoll;  #epoll（>linux2.6） kqueue(bsd)
		worker_connections 51200; #每个进程最大连接数
	}

# Config
nginx 的配置文件是分层的，比如全局配置包含 events/http , http 又包含数个server

## compress

### gzip

	gzip on;
	gzip_min_length  1100; # The length is determined only from the “Content-Length” response header field.
	gzip_types	text/plain application/x-javascript application/json text/xml text/css;
	gzip_vary on;		# Enables or disables inserting the “Vary: Accept-Encoding” response header field

## General Config
通用配置包括日志

## module

### deny ngx_http_access_module
Allows/Deny access for the specified network or address. If the special value unix: is specified (1.5.1), allows/drops access for all UNIX-domain sockets.

	Syntax:	allow/deny <address> | <CIDR> | unix: | all;
	Context:	http, server, location, limit_except

Example:

	location /.htaccess {
		deny  192.168.1.1;
		allow 192.168.1.0/24;
		allow 10.1.1.0/16;
		allow 2001:0db8::/32;
		deny all;
	}

### ngx_http_fastcgi_module
The [ngx_http_fastcgi_module](http://nginx.org/en/docs/http/ngx_http_fastcgi_module.html) module allows passing requests to a FastCGI server.

	location = / {
		fastcgi_index index.php;

		fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;

		fastcgi_param QUERY_STRING    $query_string;
		fastcgi_param REQUEST_METHOD  $request_method;
		fastcgi_param CONTENT_TYPE    $content_type;
		fastcgi_param CONTENT_LENGTH  $content_length;
		include fastcgi_params;

		#fastcgi_pass  localhost:9000;
		fastcgi_pass   unix:/var/run/fpm.sock;//可以在后面, 也可以在前面
	}

#### fastcgi_bind
	Syntax:	fastcgi_bind address | off;
	Default:	—
	Context:	http, server, location

Makes outgoing connections to a FastCGI server originate from specified ip

#### fastcgi_read_timeout

	Syntax:	fastcgi_read_timeout time;
	Default:
	fastcgi_read_timeout 60s;
	Context:	http, server, location

Defines a timeout for reading a response from the FastCGI server. The timeout is set only between two successive read operations, not for the transmission of the whole response. If the FastCGI server does not transmit anything within this time, the connection is closed.

#### fastcgi_send_timeout
	Syntax:	fastcgi_send_timeout time;
	Default:
	fastcgi_send_timeout 60s;
	Context:	http, server, location

Sets a timeout for transmitting a request to the FastCGI server. The timeout is set only between two successive write operations, not for the transmission of the whole request. If the FastCGI server does not receive anything within this time, the connection is closed.

	Syntax:	fastcgi_split_path_info regex;
	Default:	—
	Context:	location

Defines a regular expression that captures a value for the $fastcgi_path_info variable. The regular expression should have two captures: the first becomes a value of the $fastcgi_script_name variable, the second becomes a value of the $fastcgi_path_info variable. For example, with these settings

	location ~ ^(.+\.php)(.*)$ {
		fastcgi_split_path_info       ^(.+\.php)(.*)$;
		fastcgi_param SCRIPT_FILENAME /path/to/php$fastcgi_script_name;
		fastcgi_param PATH_INFO       $fastcgi_path_info;

and the “/show.php/article/0001” request, the SCRIPT_FILENAME parameter will be equal to “/path/to/php/show.php”, and the PATH_INFO parameter will be equal to “/article/0001”.

#### fastcgi_param

	Syntax:	fastcgi_param parameter value [if_not_empty];
	Default:	—
	Context:	http, server, location

Sets a parameter that should be passed to the FastCGI server. The value can contain text, variables, and their combination. These directives are inherited from the previous level if and *only if there are no fastcgi_param directives defined on the current level*.

The following example shows the minimum required settings for PHP:

	fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
	fastcgi_param QUERY_STRING    $query_string;
	The SCRIPT_FILENAME parameter is used in PHP for determining the script name, and the QUERY_STRING parameter is used to pass request parameters.

For scripts that process POST requests, the following three parameters are also required:

	fastcgi_param REQUEST_METHOD  $request_method;
	fastcgi_param CONTENT_TYPE    $content_type;
	fastcgi_param CONTENT_LENGTH  $content_length;

If PHP was built with the --enable-force-cgi-redirect configuration parameter, the REDIRECT_STATUS parameter should also be passed with the value “200”:

	fastcgi_param REDIRECT_STATUS 200;

If a directive is specified with if_not_empty (1.1.11) then such a parameter will not be passed to the server until its value is not empty:

	fastcgi_param HTTPS           $https if_not_empty;

#### fastcgi_buffer_size
	Syntax:	fastcgi_buffer_size size;
	Default:
	fastcgi_buffer_size 4k|8k;
	Context:	http, server, location

Sets the size of buffer used for reading the first part of the response received from the FastCGI server. This part usually contains a small response headers.
By default, the size is equal to the size of one buffer set by fastcgi_buffers diretive. It can be made smaller, however.

#### fastcgi_buffering
	Syntax:	fastcgi_buffering on | off;
	Default:
	fastcgi_buffering on;
	Context:	http, server, location
	This directive appeared in version 1.5.6.

Enables or disables buffering of responses from the FastCGI server.

When buffering is enabled, nginx receives respons from FastCGI server as soon as possible, saving it into buffer(set by fastcgi_buffers and fastcig_buffer_size). If responses exceed the buffer size, a part of it can be saved to a temporary file on disk(Writting to temporary file is controlled by the fastcig_max_temp_file_size and fastcgi_temp_file_write_size diretives).

When buffering is disabled, the respones is passed to a client synchronously, immediately as it is received. Note, the maximum size of the data that nginx can receive from the server at a time is set by the fastcgi_buffer_size directive.

#### fastcgi_buffers
	Syntax:	fastcgi_buffers number size;
	Default:
	fastcgi_buffers 8 4k|8k;
	Context:	http, server, location

Sets the number and size of the buffers used for reading a response from the FastCGI server, for a single connection. By default, the buffer size is equal to one memory page. This is either 4K or 8K, depending on a platform.

#### fastcgi_busy_buffers_size
	Syntax:	fastcgi_busy_buffers_size size;
	Default:
	fastcgi_busy_buffers_size 8k|16k;
	Context:	http, server, location

When buffering of responses from the FastCGI server is enabled, limits the total size of buffers that can be busy sending a response to the client while the response is not yet fully read. In the meantime, the rest of the buffers can be used for reading the response and, if needed, buffering part of the response to a temporary file. By default, size is limited by the size of two buffers set by the fastcgi_buffer_size and fastcgi_buffers directives.

#### fastcgi_connect_timeout
	Syntax:	fastcgi_connect_timeout time;
	Default:
	fastcgi_connect_timeout 60s;
	Context:	http, server, location

Defines a timeout for establishing a connection with a FastCGI server. It should be noted that this timeout cannot usually exceed 75 seconds.

#### fastcgi_hide_header
	Syntax:	fastcgi_hide_header field;
	Default:	—
	Context:	http, server, location

By default, nginx does not pass the header fields “Status” and “X-Accel-...” from the response of a FastCGI server to a client. The fastcgi_hide_header directive sets additional fields that will not be passed. If, on the contrary, the passing of fields needs to be permitted, the fastcgi_pass_header directive can be used.

#### fastcgi_ignore_client_abort

	Syntax:	fastcgi_ignore_client_abort on | off;
	Default:
	fastcgi_ignore_client_abort off;
	Context:	http, server, location

Determines whether the connection with a FastCGI server should be closed when a client closes the connection without waiting for a response.

#### fastcgi_index
	Syntax:	fastcgi_index name;
	Default:	—
	Context:	http, server, location

Sets a file name that will be appended after a URI that ends with a slash, in the value of the $fastcgi_script_name variable. For example, with these settings

	fastcgi_index index.php;
	fastcgi_param SCRIPT_FILENAME /home/www/scripts/php$fastcgi_script_name;

and the “/page.php” request, the SCRIPT_FILENAME parameter will be equal to “/home/www/scripts/php/page.php”, and with the “/” request it will be equal to “/home/www/scripts/php/index.php”.

#### fastcgi_keep_conn
	Syntax:	fastcgi_keep_conn on | off;
	Default:
	fastcgi_keep_conn off;
	Context:	http, server, location

By default, a FastCGI server will close a connection right after sending the response. However, when this directive is set to the value on, nginx will instruct a FastCGI server to keep connections open. This is necessary, in particular, for keepalive connections to FastCGI servers to function.

#### fastcgi_max_temp_file_size
	Syntax:	fastcgi_max_temp_file_size size;
	Default:
	fastcgi_max_temp_file_size 1024m;
	Context:	http, server, location

When buffering of responses from the FastCGI server is enabled, and the whole response does not fit into the buffers set by the fastcgi_buffer_size and fastcgi_buffers directives, a part of the response can be saved to a temporary file. This directive sets the maximum size of the temporary file. The size of data written to the temporary file at a time is set by the fastcgi_temp_file_write_size directive.

#### fastcgi_next_upstream

	Syntax:	fastcgi_next_upstream error | timeout | invalid_header | http_500 | http_503 | http_403 | http_404 | off ...;
	Default:
	fastcgi_next_upstream error timeout;
	Context:	http, server, location

Specifies in which cases a request should be passed to the next server:

- error
an error occurred while establishing a connection with the server, passing a request to it, or reading the response header;
- timeout
a timeout has occurred while establishing a connection with the server, passing a request to it, or reading the response header;
- invalid_header
a server returned an empty or invalid response;
- http_500
a server returned a response with the code 500;
- http_503
a server returned a response with the code 503;
- http_403
a server returned a response with the code 403;
- http_404
a server returned a response with the code 404;
- off
disables passing a request to the next server.

One should bear in mind that passing a request to the next server is only possible if nothing has been sent to a client yet. That is, if an error or timeout occurs in the middle of the transferring of a response, fixing this is impossible.

The directive also defines what is considered an unsuccessful attempt of communication with a server. The cases of error, timeout and invalid_header are always considered unsuccessful attempts, even if they are not specified in the directive. The cases of http_500 and http_503 are considered unsuccessful attempts only if they are specified in the directive. The cases of http_403 and http_404 are never considered unsuccessful attempts.

Passing a request to the next server can be limited by the number of tries and by time.

#### fastcgi_next_upstream_timeout
	Syntax:	fastcgi_next_upstream_timeout time;
	Default:
	fastcgi_next_upstream_timeout 0;
	Context:	http, server, location

This directive appeared in version 1.7.5.
Limits the time allowed to pass a request to the next server. The 0 value turns off this limitation.

#### fastcgi_next_upstream_tries
	Syntax:	fastcgi_next_upstream_tries number;
	Default:
	fastcgi_next_upstream_tries 0;
	Context:	http, server, location

Limits the number of possible tries for passing a request to the next server. The 0 value turns off this limitation.

#### fastcgi_no_cache
	Syntax:	fastcgi_no_cache string ...;
	Default:	—
	Context:	http, server, location

Defines conditions under which the response will not be saved to a cache. If at least one value of the string parameters is not empty and is not equal to “0” then the response will not be saved:

	fastcgi_no_cache $cookie_nocache $arg_nocache$arg_comment;
	fastcgi_no_cache $http_pragma    $http_authorization;

Can be used along with the fastcgi_cache_bypass directive.

#### fastcgi_cache
	Syntax:	fastcgi_cache zone | off;
	Default:
	fastcgi_cache off;
	Context:	http, server, location

Defines a shared memory zone used for caching. The same zone can be used in several places. The off parameter disables caching inherited from the previous configuration level

#### fastcgi_cache_bypass

	Syntax:	fastcgi_cache_bypass string ...;
	Default:	—
	Context:	http, server, location

Defines conditions under which the response will not be taken from a cache. If at least one value of the string parameters is not empty and is not equal to “0” then the response will not be taken from the cache:

	fastcgi_cache_bypass $cookie_nocache $arg_nocache$arg_comment;
	fastcgi_cache_bypass $http_pragma    $http_authorization;

Can be used along with the fastcgi_no_cache directive.

#### fastcgi_cache_key
	Syntax:	fastcgi_cache_key string;
	Default:	—
	Context:	http, server, location

Defines a key for caching.

#### fastcgi_cache_method
	Syntax:	fastcgi_cache_methods GET | HEAD | POST ...;
	Default:
	fastcgi_cache_methods GET HEAD;
	Context:	http, server, location

If the client request method is listed in this directive then the response will be cached. “GET” and “HEAD” methods are always added to the list, though it is recommended to specify them explicitly. See also the fastcgi_no_cache directive.

#### fastcgi_cache_path
	Syntax:	fastcgi_cache_path path [levels=levels] keys_zone=name:size [inactive=time] [max_size=size] [loader_files=number] [loader_sleep=time] [loader_threshold=time];
	Default:	—
	Context:	http

Sets the path and other parameters of a cache. Cache data are stored in files. Both the key and file name in a cache are a result of applying the MD5 function to the proxied URL. The levels parameter defines hierarchy levels of a cache. For example, in the following configuration

	fastcgi_cache_path /data/nginx/cache levels=1:2 keys_zone=one:10m;

file names in a cache will look like this:

	/data/nginx/cache/c/29/b7f54b2df7773722d382f4809d65029c

#### fastcgi_cache_valid

	Syntax:	fastcgi_cache_valid [code ...] time;
	Default:	—
	Context:	http, server, location

Sets caching time for different response codes. For example, the following directives

	fastcgi_cache_valid 200 302 10m;
	fastcgi_cache_valid 404      1m;

set 10 minutes of caching for responses with codes 200 and 302 and 1 minute for responses with code 404.

If only caching time is specified

	fastcgi_cache_valid 5m;

then only 200, 301, and 302 responses are cached.

In addition, the any parameter can be specified to cache any responses:

	fastcgi_cache_valid 200 302 10m;
	fastcgi_cache_valid 301      1h;
	fastcgi_cache_valid any      1m;

Parameters of caching can also be set directly in the response header. This has higher priority than setting of caching time using the directive. The “X-Accel-Expires” header field sets caching time of a response in seconds. The zero value disables caching for a response. If the value starts with the @ prefix, it sets an absolute time in seconds since Epoch, up to which the response may be cached. If the header does not include the “X-Accel-Expires” field, parameters of caching may be set in the header fields “Expires” or “Cache-Control”. If the header includes the “Set-Cookie” field, such a response will not be cached. Processing of one or more of these response header fields can be disabled using the fastcgi_ignore_headers directive.


### ngx_http_rewrite_module
The rewrite moude contain these directives: break, if, return, rewrite, rewrite_log, set and so on.
[rewrite](http://nginx.org/en/docs/http/ngx_http_rewrite_module.html)

#### break
	Syntax:	break;
	Default:	—
	Context:	server, location, if

*Stops* processing the current set of *ngx_http_rewrite_module directives*,

If a directive is specified *inside the location*, further processing of the request *continues* in this location.
Eg:

	if ($slow) {
		limit_rate 10k;
		break;#break No.1
	}
	location ~ / {
		#directives inside location is not influenced by break No.1, but break No.2. ;
		if($http_user_agent ~* chrome){
			break;#break No.2
		}
	}

#### if

	Syntax:	if (condition) { ... }
	Default:	—
	Context:	server, location
	Note: There is a space between if and (

If condition is evaluated as true, the *moudle directives* specified *inside the braces* are executed.

Configurations inside the if directives are inherited from the previous configuration level.

A condition may be any of the following:

- a variable name; 	false if the value of a variable is an empty string or “0”;
- -f	checking of a file existence with the “-f” and “!-f” operators;
- -d 	checking of a directory existence with the “-d” and “!-d” operators;
- -e	checking of a file, directory, or symbolic link existence with "-e" or "!-e" operators;
- -x	checking of an excutive file with "-x" or "!-x" operators;
- ~		matching of a variable against a regular expression using the "~"(for case sensitive operation) or "~*" (for case insentitive). ""
		Negative operators "!~" and "!~*" are also avaliable.
		If regular expression inludes the "}" or "{", the whole regular expression should be enclosed in single or double qoutes.
- = 	matching of a variable against a string

Example0 : check variable existence

	if ($dir = false) {
		set $dir "";
	}

Example1:

	if ($request_method = POST) {
		return 405;
	}

Example2:

	if ($fastcgi_script_name !~ ".php$"){
	}

Equal to

	location ~ ".php$"{
	}

> http://wiki.nginx.org/IfIsEvil saied that:  "if" is part of rewrite module which evaluates instructions imperatively. You must use it with fully test, if there are non-rewrite module directives inside "if"

Static Example:


	if (-f $request_filename){
		rewrite ^/(.*) /static.php/$1 break;
	}
	location ~ /static.php/(.*){
		rewrite ^/static.php/(.*) /$1 break;
	}

##### Multiple if
Refer: https://gist.github.com/Coopeh/4637216

	set $posting 0; # Make sure to declare it first to stop any warnings

	if ($request_method = POST) { # Check if request method is POST
	  set $posting N; # Initially set the $posting variable as N
	}

	if ($geoip_country_code ~ (BR|CN|KR|RU|UA) ) { # Here we're using the Nginx GeoIP module to block some spammy countries
	  set $posting "${posting}O"; # Set the $posting variable to itself plus the letter O
	}

	if ($posting = NO) { # We're looking if both of the above rules are set to spell NO
	  return 403; # If it is then let's block them!
	}

#### return

	Syntax:	return code [text];
	return code URL;
	return URL;
	Default:	—
	Context:	server, location, if

Stops processing and returns the specified code to a client. The non-standard code 444 closes a connection without sending a response header.

#### rewrite
Matches a `request URI`
如果rewrite 与location 同级，无论rewrite 是否在location 前后，rewrite 都优先执行. location 会对rewrite 后的地址做路由

	Syntax:	rewrite regex replacement [flag];
	Default:	—
	Context:	server, location, if

`rewrite` is used to modify `request URI` which is matched against specified Regular Expression(only the PATH part), it is changed as specified in `replacement` string.

- Old `QUERY_STRING` will be appended with new query in new `request URI`.
- The ` $request_uri REQUEST_URI` self will be keeped unchanged. We generally coustom `SCRIPT_URL` to store old `request_uri`
- While others is changed, such as

	`$fastcgi_script_name`
	`$script_name`
	`$document_uri`
	`$uri`
	`PHP_SELF`

> Many people set `SCRIPT_URL` with new value of `SCRIPT_NAME`
> Like `rewrite`, `try_files` has the same behaves.

    try_files $uri $uri/ /index.php?$query_string;
		先检查$uri 再检查$uri/目录 如果都不存在，则rewrite 到 /index.php?$query_string last; (Notice: internal redirection cycle)

A option 'flag' parameter can be one of:

- `last`
		stops processing the current set `ngx_http_rewrite_module` directives and *restarts*  `all locations matching with the changed URI`
- `break`
		stops processing the current set `ngx_http_rewrite_module` directives as with break directive.

- `redirect`
	returns a temporary redirect with the 302 code; Usually used *if a replacement string does not start with "http://" or "https://"*
- `permanent`
	returns a permanent redirect with the 301 code.

If query URI is '/XX.html',

Example 1: The final URI is '/mac/index.html'

	 location ~ "^/XX" {
		 rewrite "(?i)^/xx" /mac/index.html break;
	 }
	 location ~ "^/mac" {
		 rewrite "(?i)^/mac" /last.html break;
	 }

Example 2: The final URI is 'last.html'


	 location ~ "^/XX" {
		 rewrite "(?i)^/xx" /mac/index.html last;
	 }
	 location ~ "^/mac" {
		 rewrite "(?i)^/mac" /last.html break;
	 }

Example 3: The final URI is 'XX.html' (第一个location 生效, 除非里面有 rewrite last)


	 location ~ "^/XX" {
	 }
	 location ~* "^/XX" {
		 rewrite "(?i)^/XX" /last.html last;
	 }

Example 4: Ignore case

	# rewrite 忽略大小写则需要这么写
	"(?i)^/p/$"

Example 5: Preg and sub pattern 正则与子模式

	rewrite "^/(\w+)/(\w+)$" /$2/$1 last;

Example 6: the last will cause rematch all locations, so you get `final.php` to be excuted.

	 location ~ "^/a.php" {
		rewrite "^/a.php" /final.php break;
		fastcgi_pass   unix:/var/run/fpm.sock;
		fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
		include        fastcgi_params;
	 }
	 location ~* "^/XX" {
		 rewrite "^/XX" /a.php last;
	 }

第二句不可少！！

	rewrite "^/(.*)" /index.php/$1 break;
	fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;# 不能少!!!! 否则是空的200 OK. 不会执行Php


##### 500 Internal Server Error
其中一个原因是location 中的rewrite 路径一样， 在以下配置下，如查访问'/same_path', 就会报500.

因为rewrite last 会重新匹配所有的location，这就导致了`cycle location matching`:

	location / {
		rewrite /(.*) /same_path last;
	}

此时，可以通过rewrite break 避免:

	location / {
		rewrite /(.*) /same_path break;
	}

If a regular expression includes the “}” or “;” characters, the whole expressions should be enclosed in single or double quotes.

#### rewrite_log

	Syntax:	rewrite_log on | off;
	Default:
	rewrite_log off;
	Context:	http, server, location, if

Logging of `ngx_http_rewrite_module` module diretives processing result into the error_log at Notice level.

#### set
Set variable's value:

	Syntax:	set $variable value;
	Default:	—
	Context:	server, location, if

Example:

	set $script_uri $1;
	set $script_uri "${document_root}public/index.php";

## ngx_core_module
Refer to [ngx_core_module](http://nginx.org/en/docs/ngx_core_module.html)

### error_log

	Syntax:	error_log file | stderr | syslog:server=address[,parameter=value] [debug | info | notice | warn | error | crit | alert | emerg];
	Default:
	error_log logs/error.log error;
	Context:	main, http, server, location

Example:

	error_log /var/log/nginx-error.log info;
	error_log  /dir/error.log  main;
    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

> 默认目录： /usr/share/nginx/logs/error.log

### access_log

	access_log  logs/access.log  log_format;

For log format, [refer to](http://nginx.org/en/docs/http/ngx_http_log_module.html).

可以通过`nginx -V` 得到编译nginx时所默认的log 路径:

	$ nginx -V
	--http-log-path=/usr/local/var/log/nginx/access.log
	--error-log-path=/usr/local/var/log/nginx/error.log

更有效的方法是你使用：

	ps aux|grep nginx
	sudo lsof -p <master_pid>
	sudo lsof -p `ps aux |grep nginx | grep master | awk '{print $2}'` | grep log

### env

	Syntax:	env variable[=value];
	Default:
		env TZ;
	Context:	main

By default, *nginx removes all environment variables inherited from its parent process* except the TZ variable. This directive allows preserving some of the inherited variables, changing their values, or creating new environment variables. These variables are then:

	env PATH=.

#### SERVER ENV
在apache 下的SERVER env

	<IfModule env_module>
		SetEnv APP_ENV 1
		SetEnv DEBUG 1
	</IfModule>

nginx 与apache 机制不一样：

	# php-fpm
	env[APP_ENV] = production

	# php-cgi
	location / {
	    fastcgi_param APP_ENV production;
	}

如果想在命令行下传SERVER 变量:

	DEBUG=debug var1=var php -r 'echo $_SERVER["DEBUG"],$_SERVER["var1"]; '

### include

	include mime.types;
	include vhosts/*.conf;

### use

	Syntax:	use method;
	Default:	—
	Context:	events

Specifies the [connection processing method](http://nginx.org/en/docs/events.html) (such as epoll on linux or kqueue on MacOSX) to use. There is normally no need to specify it explicitly, because nginx will by default use the most efficient method.

### user
	Syntax:	user user [group];
	Default:
	user nobody nobody;
	Context:	main

Defines user and group credentials used by worker processes. If `group` is omitted, a group same as `user` is used.
If group is not existed, it would create an error `nginx: [emerg] getgrnam(group)  failed in`

### worker_aio_requests
	Syntax:	worker_aio_requests number;
	Default:
	worker_aio_requests 32;
	Context:	events

When using aio with the epoll connection processing method, sets the maximum number of outstanding *asynchronous I/O operations* for a single worker process.

### worker_connection
	Syntax:	worker_connections number;
	Default:
	worker_connections 512;
	Context:	events

Sets the maximum number of *simultaneous(同时，并发) connections* that opened opened by a worker process.

Another consideration is that the actual number of simultaneous connections cannot exceed current limit on maximum number open files. which can be changed by `worker_rlimit_nofile`.

### worker_process
	Syntax:	worker_processes number | auto;
	Default:
	worker_processes 1;
	Context:	main

Defines the number of worker process.

The optimal value depends on many factors including the number of CPU cores, the number of hard disk drives , and load pattern. When one is doubt, set it to available CPU cores would be a good start(the value “auto” will try to autodetect it).

### worker_cpu_affinity
	Syntax:	worker_cpu_affinity cpumask ...;
	Default:	—
	Context:	main

Bind worker processes to the sets of CPUs. Each CPU set if represent by a bitmask of allowed CPUs.

Example:

	worker_processes    4;
	worker_cpu_affinity 0001 0010 0100 1000;

### worker_priority

	Syntax:	worker_priority number;
	Default:
	worker_priority 0;
	Context:	main

Defines the scheduling priority of worker processes like it is done by the nice command: a negative means a higher priority. Allowed ranges normally from -20 to 20.

## http_proxy
The ngx_http_proxy_module module allows passing requests to another server.

### proxy_pass

	Syntax:	proxy_pass URL;
	Default:	—
	Context:	location, if in location, limit_except

Sets the protocol and address of a proxied server and an `optional URI` to which a location should be mapped.
As a protocol, “http” or “https” can be specified.
The address can be specified as a domain name or IP address, and an optional port:

	proxy_pass http://localhost:8000/uri/;

or as a UNIX-domain socket path specified after the word “unix” and enclosed in colons:

	proxy_pass http://unix:/tmp/backend.socket:/uri/;

If a domain name resolves to several addresses, all of them will be used in a round-robin fashion. In addition, an address can be specified as a server group.

#### proxy host
代理时, 记得传递host：

	proxy_set_header Host $http_host; # 127.0.0.1:8589
	# proxy_set_header Host $host; # 127.0.0.1
	proxy_set_header X-Real-IP  $remote_addr;
	proxy_set_header X-Forwarded-For $remote_addr;

##### restricting access
https://www.nginx.com/resources/admin-guide/restricting-access/

ip limit

	location /{
		allow 192.168.1.1/24;
		allow 127.0.0.1;
		deny 192.168.1.2;
		deny all;
	}

For proxy `proxy_set_header X-Forwarded-For $remote_addr;`

	set $allow false;
	if ($remote_addr ~ " ?127\.0\.0\.1$") {
	if ($http_x_forwarded_for ~ " ?127\.0\.0\.1$") {
		set $allow true;
	}
	if ($allow = false) {
		return 403;
	}

#### proxy_pass URI
A request URI is passed to the server as follows:

- If the proxy_pass is with a URI, the part of a *normalized request URI*(literal location only) matching the location is replaced by a URI specified in the directive:

	location /name/ {
		proxy_pass http://127.0.0.1/remote/;//access "http://host/name/act" will be replaced with "http://host/remote/act"
	}

- If proxy_pass is specified without a URI,
the request URI is passed to the server in the *same form* as sent by a client when the original request is processed

    proxy_pass http://127.0.0.1;
    proxy_pass http://api.hilojack.com;

> When location is specified using a regular expression,  the directive should be specified `without a URI`

#### Variable
If you use variables in `proxy_pass`, nginx will not use local `/etc/hosts` and `local dns setting`
You should specify `resolver` instead (in case of proxy loop).

	resolver 223.5.5.5;
	#proxy_pass $scheme://$http_host/$request_uri;
	proxy_pass $scheme://$http_host;

`resolver` will not use `hosts` file.  You can get around this by installing `dnsmasq` and setting your resolver to 127.0.0.1.
Basically this uses your local DNS as a resolver, but it only resolves what it knows about (among those things is your /etc/hosts) and forwards the rest to your default DNS.
> But sadly dnsmasq does not automatically detect changes in hosts file. You have to send HUP

	sudo yum install dnsmasq -y;
	service dnsmasq start;
	sudo chkconfig --level 235 dnsmasq on

#### proxy
example

	server {
		listen 8888;
		location / {
			resolver 223.5.5.5;
			if ( $host ~ "hilojack.com" ) {
				proxy_pass http://127.0.0.1:80;
			}
			if ( $host !~ "hilojack.com" ) {
				proxy_pass $scheme://$http_host;
			}
		}
	}

curl -x '127.0.0.1:8888' 'http://hilojack.com/test.php'

## http

### keepalive_timeout

	Syntax:	keepalive_timeout timeout [header_timeout];
	Default:
	keepalive_timeout 75s;
	Context:	http, server, location

	keepalive_timeout 0;//disable timeout

### server_name

	listen 80;
	server_name host1 host2 "";
	server_name "";//empty host
	server_name _;// catch-all invalid name

The server_name will be matched in following order of precedence:

1. exact name
2. *longest wildcard name* starting/ending with an asterisk, e.g. `"*.org", "mail.*"`. These name are invalid `"www.*.hilojack.com"`, `"hilo*.com"`
2. *a special wildcard name* ".hilojack.com" imply both `"hilojack.com"` and `"*.hilojack.com"` .
3. *Regular expression names* must start with the tilde character, e.g. `"~^(?<name>\d{1,3}+)\.hilo\.net$"`.

Note: A expressioin name contains character "{}" should be quoted.

	server_name   "~^(\w+)\.(?<domain>\w{1,3}.+)\.com$";
	location / {
		root   /sites/$domain/$1;
	}

Refer to:
http://nginx.org/en/docs/http/server_names.html

如果同时监听https/http 端口, 或者多个端口, 需要设定default_server 的port：

	listen       80;
	listen 443 default_server ssl;#加ssl 时会自动开启ssl, 不能再加 ssl on;

### upstream
定义一组服务器， UNIX/TCP 可以 混合使用

	语法:	upstream name { ... }
	默认值:	—
	上下文:	http

upstream目前支持 5 种方式的分配

	1)、轮询（默认）
	每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除。
	2)、weight
	指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况。
	3)、ip_hash
	每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session的问题。
	4)、fair（第三方）
	按后端服务器的响应时间来分配请求，响应时间短的优先分配。
	5)、url_hash（第三方）
	按访问的url的hash结果分配，使每个url定向到同一个后端服务器，后端为缓存服务器比较有效。


Example:

	upstream backend {
		server backup1.example.com:8080;
		server backup2.example.com:8080 down;
		server backup3.example.com:8080 weight=2;
		server backup4.example.com:8080 ip_hash;
		server backup5.example.com:8080 url_hash;
		server backup6.example.com:8080 fair; //time
		server backup7.example.com:8080 backup;

        server 192.168.0.101 max_fails=2 fail_timeout=30s;
		server unix:/tmp/backend3;

		keepalive 8;// maximum number of idle keepalive connections to upstream servers that are preserved in the cache of each worker process.
	}
	server {
		location / {
			proxy_pass http://backend;
		}
	}


upstream 每个设备的状态:

	down 表示单前的server暂时不参与负载
	weight 默认为1.weight越大，负载的权重就越大。
	max_fails ：允许请求失败的次数默认为1.当超过最大次数时，返回proxy_next_upstream 模块定义的错误
	fail_timeout:max_fails 次失败后，暂停的时间。
	backup： 其它所有的非backup机器down或者忙的时候，请求backup机器。所以这台机器压力会最轻。

#### memcache upstream

	upstream memcached_backend {
		server 127.0.0.1:11211;
		server 10.0.0.2:11211;
		 keepalive 32;
	}

	server {
		location /memcached/ {
			set $memcached_key $uri;
			memcached_pass memcached_backend;
		}
	}

#### fastcgi keepalive(fastcgi_pass)
启用fastcgi 长连接支持 除了需要在upstream 中配置 `keepalive N` 外，还需要在 location 中增加 `fastcgi_keep_conn on`

	upstream fastcgi_backend {
		server 127.0.0.1:9000;
		 keepalive 8;
	}

	server {
		location /fastcgi/ {
			fastcgi_pass fastcgi_backend;
			fastcgi_keep_conn on;
			...
		}
	}

#### http keepalive(proxy_pass)
启用对后端机器HTTP 长连接支持
只有 http1.1 才支持 keepalive, 所以要传一个版本1.1 请求头

	upstream http_backend {
		server 127.0.0.1:8080;
		keepalive 16;
	}

	server {
		...
		location /http/ {
			proxy_pass http://http_backend;
			proxy_http_version 1.1;
			proxy_set_header Connection "";
			...
		}
	}

### proxy

#### location 路由语句块
Sets configuration depending on a `request URI`.

location 是用于路由的. 默认路由 或者location 语句体为空，则直接去读静态资源

Syntax of location：

	Syntax: location [ = | ~ | ~* | ^~ ] uri { ... }
	location { } @ name { ... }
	Context: server / location

	location regExp{ }

Case sensitive for location on MacOSX:

	../configure --with-cc-opt="-D NGX_HAVE_CASELESS_FILESYSTEM=0"

#### 优先级

	~      #波浪线表示执行一个正则匹配，区分大小写
	~*    #表示执行一个正则匹配，不区分大小写
	^~    #^~表示普通字符前缀匹配，相当于没有`^~`
	=     #进行普通字符精确匹配
	@     #"@" 定义一个命名的 location，使用在内部定向时，例如 error_page, try_files

优先级(location 是顺序无关的):

	= 精确匹配会第一个被处理。如果发现精确匹配，nginx停止搜索其他匹配。
	正则表达式(本身按顺序)
	长普通字符匹配
	短普通字符匹配

##### with regular expression
In order to use regular expressions, you must always use a prefix:

	"~"  must be used for case sensitive matching
	"~*" must be used for case insensitive matching

	# 区分大小写
	location ~ "regExp" { }
	# 忽略大小写
	location ~* "regExp" { }

Example:

	location ~ "\.png$" {
		#mathes any query ending with png.
	}

Note: Nginx has the ability to decode URIs in real time. For example, in order to match “/app/%20/images” you may use “/app/ /images” to determine the location.

##### with literal strings

With solid literal strings

	# matches only the "/literal$" query, "$" is a literal char.
	location = "^/literal$$" { }

With temporary literal strings(`^~` 可忽略)

	# Note: it will continue search next localtion with and this match will be valid only if it there is *no other more characters matches*
	# matches any query that start with "/literal$", note "$" is a literal char.
	location "/literal$" { }

# header

## add_header

	add_header Access-Control-Allow-Origin *
	context: server, location

# compress

	gzip on;
	context: http, server

# fastcgi_pass

	Context:	location, if in location

Sets the address of a FastCGI server. The address can be specified as a domain name or IP address, and an optional port:

	fastcgi_pass localhost:9000;

or as a UNIX-domain socket path:

	fastcgi_pass unix:/tmp/fastcgi.socket;
	fastcgi_param   SCRIPT_FILENAME    "${document_root}/public/index.php";
	include fastcgi_params;

# ssl(https)

	server {
		listen 80;
		listen 443 default_server ssl;#加ssl 时会自动开启ssl, 不能再加 ssl on;

		#ssl on;
		#listen 443 ssl;

		ssl_certificate /Users/hilojack/test/ssl/my.crt;			#cert.pem cert.crt
		ssl_certificate_key /Users/hilojack/test/ssl/my_nopass.key; #cert.key
		# 若ssl_certificate_key使用my.key，则每次启动Nginx服务器都要求输入key的密码。

		#ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;#defalut
		#ssl_ciphers         HIGH:!aNULL:!MD5; #default
		#ssl_prefer_server_ciphers  off;#default off, Specifies that server ciphers should be preferred over client ciphers

		#ssl_session_cache    shared:SSL:1m;
		#ssl_session_timeout  5m;

	}

Refer to [](/p/aogorithm-ssl) and [nginx_https]

安全建议：( Refer: https://www.linux.cn/article-5374-1.html)

1. 不要使用存在安全缺陷的SSLv3 及以下协议, 并设置更强壮的加密套件（cipher suite）来尽可能启用前向安全性(Forward Secrecy)
2. 禁用SSL 压缩来隐低CRIME 攻击

> 免费获得非商业证书见: http://www.startssl.com/

# tcp

	location /video/ {
		sendfile       on;#default off
		tcp_nopush     on;#default off
		aio            on;
	}

## aio
Enables or disables the use of asynchronous file I/O (AIO) on FreeBSD and Linux:

	location /video/ {
		aio on;
		output_buffers 1 64k;
	}

## sendfile
传统网络传输过程：

	read(file,tmp_buf, len);
	write(socket,tmp_buf, len);
	硬盘 >> kernel buffer >> user buffer>> kernel socket buffer >>协议栈

用`sendfile()` 来进行网络传输的过程：

	sendfile(socket,file, len);
	硬盘 >> kernel buffer (快速拷贝到kernelsocket buffer) >>协议栈

Refer: http://blog.csdn.net/zmj_88888888/article/details/9169227

## tcp_nopush
*tcp_nopush*
	option will make nginx to send all header files in a single packet rather than seperate packets.(when sendfile on)

Enables or disables the use of the `TCP_NOPUSH socket` option on FreeBSD or the `TCP_CORK socket` option on Linux.
The options are enabled only when sendfile is used. Enabling the option allows

1. sending the response header and the beginning of a file in one packet, on Linux and FreeBSD 4.*;
1. sending a file in full packets.

`tcp_nopush = on` 时执行系统调用 `tcp_cork()` ，结果就是数据包不会马上传送出去，等到数据包最大时，一次性的传输出去，这样有助于解决网络堵塞。

tcp_nopush 基本上控制了包的“Nagle化”，Nagle化在这里的含义是采用Nagle算法把较小的包组装为更大的帧。

## tcp_nodelay
*tcp_nodelay* 与nopush 相反，不会同时生效
	Enables or disables the use of the TCP_NODELAY option. The option is enabled only when a connection is transitioned into the keep-alive state.(nodelay, keep-alive 多次传输不需要网络连接)

# todo
https://github.com/taobao/nginx-book

# Reference
- [nginx performance]
- [https]

[nginx performance]: http://nginx.com/blog/tuning-nginx/
[nginx_https]: http://nginx.org/cn/docs/http/configuring_https_servers.html
