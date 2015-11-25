---
layout: page
title:
category: blog
description:
---
# Preface

# ajax 串行
TODO:
http://www.2fz1.com/post/javascript-yi-bu-bian-cheng/

# ajax

	var xhr=new XMLHttpRequest();
	xhr.onreadystatechange=function() {
		if (xhr.readyState==4 && xhr.status==200) {
			var a=xhr.responseText;
			console.log(a);
		}
	}
	xhr.open("POST","http://hilo.sinaapp.com/header.php?demo",true);
	xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
	xhr.setRequestHeader('Accept', 'application/json');
	xhr.setRequestHeader('X-Requested-With', 'XMLHttpRequest');
	xhr.send("xuehui1=1&xuehui2=2");

Detect Ajax：

	$_SERVER['HTTP_X_REQUESTED_WITH']
	$_SERVER['HTTP_ACCEPT'] === 'application/json';

默认的`Content-Type:text/plain + POST `只会传`RAW_POST_DATA` , `application/x-www-form-urlencode` 才会传`$_POST`, `enctype="multipart/form-data"` 则包括`POST+FILES`

	$GLOBALS['HTTP_RAW_POST_DATA']

## ajax jquery
For html5, emulate jquery ajax

	function request(type, url, opts, callback) {
	　　var xhr = new XMLHttpRequest();
	　　if (typeof opts === 'function') {
	　　　　callback = opts;
	　　　　opts = null;
	　　}
	　　xhr.open(type, url);
	　　var fd = new FormData();
	　　if (type === 'POST' && opts) {
	　　　　for (var key in opts) {
	　　　　　　fd.append(key, JSON.stringify(opts[key]));
	　　　　}
	　　}
	　　xhr.onload = function () {
	　　　　callback(JSON.parse(xhr.response));
	　　};
	　　xhr.send(opts ? fd : null);
	}

然后，基于request函数，模拟jQuery的get和post方法。

	var get = request.bind(this, 'GET');
	var post = request.bind(this, 'POST');

# debug
每次请求xhr 最好新建一下xhr. 或者销毁覆盖`xhr`

	var xhr = new XMLHttpRequest();
	xhr.addEventListener('load',func);//多次addEventListener 会出问题

# listener
demo: [js-lib/chunk/](js-lib/chunk/)
refer: http://stackoverflow.com/questions/7853467/uploading-a-file-in-chunks-using-html5

	function sendRequest() {
		var blob = document.getElementById('fileToUpload').files[0];
		const BYTES_PER_CHUNK = 2000; // 2kb chunk sizes.
		const SIZE = blob.size;
		var start = 0;
		var end = BYTES_PER_CHUNK;
		while( start < SIZE ) {
			var chunk = blob.slice(start, end);
			uploadFile(chunk);
			start = end;
			end = start + BYTES_PER_CHUNK;
		}
	}

	function uploadFile(blobFile) {
		var fd = new FormData();
		fd.append("fileToUpload", blobFile);
		fd.append("var", 'value');

		var xhr = new XMLHttpRequest();
		xhr.upload.addEventListener("progress", uploadProgress, false);
		xhr.addEventListener("load", uploadComplete, false);
		xhr.addEventListener("error", uploadFailed, false);
		xhr.addEventListener("abort", uploadCanceled, false);
		xhr.open("POST", "upload.php");
		xhr.send(fd);
	}

	function uploadProgress(evt) {
		if (evt.lengthComputable) {
			console.log({total:evt.total, loaded:evt.loaded});
			var percentComplete = Math.round(evt.loaded * 100 / evt.total);
			document.getElementById('progressNumber').innerHTML = percentComplete.toString() + '%';
		}
		else {
			document.getElementById('progressNumber').innerHTML = 'unable to compute';
		}
	}

	function uploadComplete(evt) {
		//console.log(evt.target.responseText);
	}

	function uploadFailed(evt) {
		alert("There was an error attempting to upload the file.");
	}

	function uploadCanceled(evt) {
		xhr.abort();
		xhr = null;
	}

## Ajax upload progress bar
http://www.sitepoint.com/html5-javascript-file-upload-progress-bar/

	<progress max="100" value="0"></progress>

# AJAX FormData
> 上传文件时，不能用 xhr.setRequestHeader("Content-type", "application/x-www-form-urlencoded");
> 而必须使用默认的: Content-Type:multipart/form-data; boundary=----WebKitFormBoundaryfyRdj8roosVVWIsH

Via FormData and formnode:

	new FormData(document.getElementById());

Via FormData and file:

	var files = document.getElementById('photos').files;
	var formData = new FormData();
	for (var i = 0; i < files.length; i++) {
	  var file = files[i];
	  formData.append('photos[]', file, file.name);
	}

	// In jquery
	$.each(files, function(key, value) {
        formData.append(key, value);
    });

Add form listener:

	$('form').submit(function(event) {
        event.preventDefault();
		ajax....uploading code
    });

> here is an demo:
http://stackoverflow.com/questions/166221/how-can-i-upload-files-asynchronously

## Ajax FormData Jquery

	$.ajax({
		url: '/admin/banlance/sendbanlance?act=addTask',
		data: fd,
		contentType: false,//这两行必须加
		processData: false,//
		method: 'POST'
	}).done(function(data){})

## JSON AJAX
https://ajax.googleapis.com/ajax/services/search/images?v=1.0&q=http://m.weibo.cn

	var x = new XMLHttpRequest();
	x.open('GET', searchUrl);
	x.responseType = 'json';
	x.onload = function(){
		res = x.response;
		console.log(res);//json
	}


## CORS
> ajax 可以跨域发请求(不能带跨域的Cookie)，不过不能收响应

正统的方法是CORS(Cross-origin resource sharing), 服务端返回:

	header('Access-Control-Allow-Origin:*');
	header('Access-Control-Allow-Origin:hilojack.com');//只能跟一个域名, 如果想支持多个域名，服务端应该根据Referer 返回不同的域名
	header('Access-Control-Allow-Methods: GET, POST, PUT, DELETE, OPTIONS');
	header('Access-Control-Allow-Headers: Authorization');

Hack 方法有jsonp, 原理是通过script 标签, 发一个get 请求，请求时参数中加一个回调方法callback 用于处理请求数据。这依然需要服务端配合。

### Apache & nginx
不支持子域名`*.hilojack.com`

	# apache
	Header set Access-Control-Allow-Origin *
	Header set Access-Control-Allow-Headers Authorization

	# nginx
	add_header Access-Control-Allow-Origin *;
	add_header Access-Control-Allow-Headers Authorization;

