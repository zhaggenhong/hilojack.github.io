---
layout: page
title:
category: blog
description:
---
# Preface


# Variable Env Type

## Embedded Variables
The ngx_http_core_module module supports embedded variables with names matching the Apache Server variables. First of all, these are variables representing client request header fields, such as $http_user_agent, $http_cookie, and so on. Also there are other variables:

	http://localhost/a/b/c/?var1=val1

    #按脚本 ****************
    SCRIPT_FILENAME = DOCUMENT_ROOT + truePath
        /data1/www/htdocs/912/hilo/1/phpinfo.php
    DOCUMENT_ROOT
        /data1/www/htdocs/912/hilo/1

    #按url
    path: SCRIPT_NAME /PHP_SELF / DOCUMENT_URI (nginx: SCRIPT_URL 默认是空的)
		nginx: $fastcgi_script_name , 这可以被 rewrite 改写, 以上path 都会变
			/a/b/c/
		REQUEST_URI(query) 则不会变

    SCRIPT_URI = HTTP_HOST+path 可能为空
        http://hilojack.com/a/b/c/
    REQUEST_URI = path + QUERY_STRING
		nginx: $request_uri
        /a/b/c/?test=1

其它:

	$_SERVER['OLDPWD']
		The definition of OLDPWD is the *p*revious *w*orking *d*irectory as set by the cd command

## ENV

	fastcgi_param  QUERY_STRING       $query_string;
	fastcgi_param  REQUEST_METHOD     $request_method;
	fastcgi_param  CONTENT_TYPE       $content_type;
	fastcgi_param  CONTENT_LENGTH     $content_length;

	fastcgi_param  SCRIPT_NAME        $fastcgi_script_name;	/path "modified by rewrite. You can save it's value in SCRIPT_URL
	fastcgi_param  REQUEST_URI        $request_uri; /path?query
	fastcgi_param  DOCUMENT_URI       $document_uri;
	fastcgi_param  DOCUMENT_ROOT      $document_root;
	fastcgi_param  SERVER_PROTOCOL    $server_protocol;
	fastcgi_param  HTTPS              $https if_not_empty;

	fastcgi_param  GATEWAY_INTERFACE  CGI/1.1;
	fastcgi_param  SERVER_SOFTWARE    nginx/$nginx_version;

	fastcgi_param  REMOTE_ADDR        $remote_addr;
	fastcgi_param  REMOTE_PORT        $remote_port;

	fastcgi_param  SERVER_ADDR        $server_addr;
	fastcgi_param  SERVER_PORT        $server_port;
	fastcgi_param  SERVER_NAME        $server_name;

	fastcgi_param  REQUEST_UID       $request_uid;??

# String
Concat String

	"${document_root}public/index.php";
	'${document_root}public/index.php';//不区分单双引号

	$scheme://$http_host$request_uri
	$request GET 1.0 /test.php

## regexp
$1 是第一个子模式匹配

	if ( $request_uri ~* "([^?]*)?" ) {
			set $script_uri $1;
	}

