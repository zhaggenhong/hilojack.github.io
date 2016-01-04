---
layout: page
title:	php 知识库
category: blog
description:
---
# Preface
关于php 的函数

# todo
https://github.com/AnewG/Modern-php?files=1

# Configration
可以用parse_ini_file()函数解析INI配置文件


# Data Type

	ctype_alnum
	ctype_digit
	ctype_alpha
	in_numeric

	settype(&$var, $type);
	gettype($var);


# filter

## Validate Filter

	var_dump(filter_var('bob@example.com', FILTER_VALIDATE_EMAIL));
	var_dump(filter_var('http://example.com', FILTER_VALIDATE_URL, FILTER_FLAG_PATH_REQUIRED));//false
	FILTER_VALIDATE_IP
	FILTER_VALIDATE_FLOAT
	FILTER_VALIDATE_INT
	FILTER_VALIDATE_REGEXP

### Sanitize filters

	var_dump('<a>hello</a>', FILTER_SANITIZE_STRING); //hello, like strip_tags
	FILTER_SANITIZE_EMAIL 去掉和邮件地址无关的字符
	FILTER_SANITIZE_NUMBER_INT 去掉和INT 无关的字符
	FILTER_SANITIZE_SPECIAL_CHARS 去HTML 特殊字符 `<>&'"` 编码

# Const

	const N = 1
	get_defined_constants(true)['user']
	get_defined_constants(true)['Core']
	get_defined_constants(true)['xdebug']

# Array

## next end

	$transport = array('foot', 'bike', 'car', 'plane');
	$mode = current($transport); // $mode = 'foot';
	$mode = next($transport);    // $mode = 'bike';
	$mode = next($transport);    // $mode = 'car';
	$mode = prev($transport);    // $mode = 'bike';
	$mode = end($transport);     // $mode = 'plane';

## compact and extract

### compact
For each of these, `compact()` looks for a variable with that name in the current symbol table
and adds it to the output array such that the variable name becomes the key and the contents of the variable become the value for that key.
In short, it does the opposite of extract().

	<?php
	$out = '234';
	function a(){
		$var1 = '1';
		$var2 = '2';
		var_dump(compact('var1', ['var2', 'out']));
	}
	a();

Results:

	array(2) {
	  ["var1"]=> string(1) "1"
	  ["var2"]=> string(1) "2"
	}

### extract

	int extract ( array &$array [, int $flags = EXTR_OVERWRITE [, string $prefix = NULL ]] )
		Key as variable name, and val for that key as content of variable

	extract($var_array, EXTR_PREFIX_SAME, "prefix");

## Pointer, 指针
prev, current, next, reset, end

list($k, $v) = each($arr)

## Iterator
Interface for external iterators or objects that can be iterated themselves internally.(current/key/next/rewind/valid)

### array_keys array_values
array_keys 与 array_values 得到的元素顺序不会变
http://stackoverflow.com/questions/21092095/is-elements-order-the-same-after-array-being-split-by-array-keys-and-array-val

### walk

	//用于改变原数组 或者 通过闭包函数返回新的数组或者结果集
	bool array_walk ( array &$array , callable $callback [, mixed $userdata = NULL ] );//$callback(&$item, $key, &$userdata = null). 始终返回true. 但是外部的$userdata 始终是按值传递进去的。
	//用于生成新数组
	$b = array_map($func, $a, $b, ...);//返回一个数组：array($func($a_item,$b_item,...)); //$func 按引用传值 无效
	array_map('sum', $a, $b);

	//累积,累加,累...
	mixed array_reduce($arr, $func[, $init_value = null]); //sum($carry = $init_value, $item)  不支持传$key

### filter array
返回过滤后的数组

	array array_filter ( array $input [, callable $callback = "" ] );//1. bool $callback($v); 2. $callback 可以按引用传值

Example:

	$arr = ['a'=>1, 'b'=>0, 'c'=>false, 'd'=>''];
	var_dump(array_filter($arr));
	array(1) {
	  ["a"]=>
	  int(1)
	}

### filter array keys

	$arr = ['foo'=>'value1', 'bar' => 'value2'];
	$allowed = ['foo'];
	var_dump(array_intersect_key($arr, array_flip($allowed)));
		array(1) {
		  ["foo"]=>
		  string(6) "value1"
		}

	var_dump(array_diff_key($arr, array_flip($allowed)));
		  "bar"=> "value2"

### array_column

	$b = array_column([['a'=>1,'b'=>2], ['a'=>3,'b'=>4]],'b');
	print_r($b); //[2, 4]
	$b = array_column([['a'=>1,'b'=>2], ['a'=>3,'b'=>4]],'b', 'a');
	[1=>2, 3=>4]

## 统计频度

### array_count_values

	$array = array(1, "hello", 1, "world", "hello");
	print_r(array_count_values($array));//1=>2 hello=>2 world=>1

### array_unique
	array_unique();

## sort 排序
Trim index association

	sort($arr)
	rsort
	usort
	shuffle

Maintain index association

	asort
	ksort
	natsort

	uasort
	uksort

With no params reference:

	array_reverse
	array_flip

array_flip 跟array_merge 一样，后面的key 会覆盖前面的

	var_dump(array_flip(['a'=>'b', 'c'=>'b']));
	array(1) {
	  ["b"]=>
	  string(1) "c"
	}

## 合并-拆分

	array_combine($keys, $values)
	array_merge(["k"=>1, 2=>3], ["k"=>0, 4=>5]); //Array ( [k] => 0 [0] => 3 [1] => 5)
	["k"=>1, 2=>3]+["k"=>0, 2=>5]; //不覆盖 Array ( [k] => 1 [2] => 3)
	array_slice($arr, $start, $length, $preserve_key = false)
	array_splice

## 交差并集

### 并

	var_dump(['a'=>1]+['a'=>2]);
	array(1) {
	  ["a"]=>
	  int(1)
	}

### 交差

	array_intersect();
	array_intersect_assoc();
	array_diff(); 			# diff value only
	array_diff_assoc();		# diff value and key
	array_diff_key();		# diff key only

array_remove_key

	array_diff_key($data, array_flip($bad_keys));

array_keep_key

	function array_keep_key($data, $keep_keys) {
        $arr = [];
        foreach ($keep_keys as $k) {
            //isset($data[$k]) && $arr[$k] = $data[$k];
            array_key_exists($k, $data) && $arr[$k] = $data[$k];
        }
        return $arr;
    }
	function array_keep_key($data, $keep_keys){
		array_diff_key($data,array_flip(array_diff(array_keys($data), $keep_keys)));
	}


## rand

	array_rand($arr[, $num = 1]);//返回rand_key or rand_keys
	shuffle()

## range

	range(1, 10);//1,2,3...10

## 求和

	array_sum

## chunk(split)

	array_chunk($array, $size)

## fill & pad

	Array array_fill($start, $num, $value);
		 var_dump(array_fill(5, 2, '?'));
		array(2) {
		  [5]=>
		  string(1) "?"
		  [6]=>
		  string(1) "?"
		}
	Array array_pad($arr, $len, $value);

# Object

## instance

	/**
     * @return static
     */
    public static function instance() {
        $className = get_called_class();
        if (self::$objects[$className] === null) {
            self::$objects[$className] = new static;
        }
        return self::$objects[$className];
    }

## callback

	call_user_func_array(array(__CLASS__, 'func'), $arr);//static
	call_user_func_array(array($this, 'func'), $arr);//obj

# class

## traits
traits 是为单继承语言设计的一个代码复用机制。它和类组合的语义，避免了传统的多继承和混入类（Mixin）的复杂性。

Example:

	trait classA{
		function funA(){ echo __METHOD__ ."\n";	}
	}
	trait classB{
		function funB(){ echo __METHOD__ ."\n";	}
	}

	class A extends Exception{
		use classA;
		use classB;
		function __construct(){
			$this->funA();	//classA::funA
			$this->funB();	//classB::funB
		}
	}
	new A;

# Exception & Error

## try catch

	try{$this->clean(); return $res;}
	finally{return $this->clean();}

## Exception
Exception 用来API 错误信息返回特别有方便,
这比每个api 函数自己封装errno/err 要方便多了

	class BaseApiController{
		function __construct(){
			set_exception_handler(function($e){
				echo json_encode([
					'errno' => $e->getCode(),
					'error' => $e->getMessage(),
				], JSON_UNESCAPED_UNICODE);
			});
		}
	}

restore_exception:

	set_exception_handler('exception_handler_1');
    set_exception_handler('exception_handler_2');
    restore_exception_handler();

## Error
The following error cannot be handled with user defined function: E_ERROR, E_PARSE, E_CORE_ERROR, E_CORE_WARNING, E_COMPILE_ERROR, E_COMPILE_WARNING, and most of E_STRICT raised in the file where set_error_handler() is called.

	set_error_handler(function ($errCode, $errMesg, $errFile, $errLine) {
		echo "Error Occurred\n";
		throw new Exception($errMesg);
	});

如果要捕获Fatal Error, 可以用register_shutdown_function(), 参考[](http://stackoverflow.com/questions/277224/how-do-i-catch-a-php-fatal-error):

	register_shutdown_function( "fatal_handler" );
	function fatal_handler() {
	  $error = error_get_last();
	  if( $error !== NULL) {
		/*	$error["type"];
			$error["file"];
			$error["line"];
			$error["message"];
		 */
		  $error['errno'] = E_CORE_ERROR;
		  $trace = print_r( debug_backtrace( false ), true );
		  var_dump($trace, $error);
	  }
	}

Error handler 内throw 的异常不能被 exception handler 捕获，error handler 相当于exit 会跳过 exception handler
Refer to : [深入理解异常](http://www.laruence.com/2010/08/03/1697.html)

# fastcgi

## fastcgi_finish_request
flush数据到客户端。调用这个方法后，再有任何输出内容，都不会输出到客户端。

# Function

## Anonymous function recursively

	$f = function() use($bar, $foo, &$f) { //$f 必须为引用，否则$f 是undefined
	   $f();
	};

## get_defined_functions

	array get_defined_functions(void)['user'];
	array get_defined_functions(void)['internal'];

## function argument

	func_num_args(void),
	func_get_arg(n), // get n'th argument
	func_get_args(void).

## register_shutdown_function
register_shutdown_function 可以有多个, 在php 结束时触发：

	register_shutdown_function( func1 );
	register_shutdown_function( func2 );

但是它对die 是没有用的，不过可以使用`__destruct()` 或者`ob_start`

# Math

## rand

	rand(0,99); //Rand from 0 to 99(). It does not generate cryptographically secure values
	mt_rand(0,99); //Generater a better random integer

## pi

	echo M_PI; // 3.1415926535898

## base

	echo base_convert('4', 16, 2);//100

# XML
simpleXML

## load

	$xmlstr = <<<XML
	<?xml version='1.0' standalone='yes'?>
	<movies>
	 <movie>
	  <title>PHP: Behind the Parser</title>
	  <characters>
	   <character>
		<name>Ms. Coder</name>
		<actor>Onlivia Actora</actor>
	   </character>
	   <character>
		<name>Mr. Coder</name>
		<actor>El Act&#211;r</actor>
	   </character>
	  </characters>
	  <plot>
	   So, this language. It's like, a programming language. Or is it a
	   scripting language? All is revealed in this thrilling horror spoof
	   of a documentary.
	  </plot>
	  <great-lines>
	   <line>PHP solves all my web problems</line>
	  </great-lines>
	  <rating type="thumbs">7</rating>
	  <rating type="stars">5</rating>
	 </movie>
	</movies>
	XML;

### loadStr
	$movies = new SimpleXMLElement($xmlstr);
	//or $movies = simplexml_load_string($xmlstr);

### loadDom
	$dom = new DOMDocument;
	$dom->loadXML('<books><book><title>blah</title></book></books>');
	if (!$dom) {
		echo 'Error while parsing the document';
		exit;
	}

	$books = simplexml_import_dom($dom);

	echo $books->book[0]->title;

### asXML

	echo $movies->asXML();

## Operation

### get
	echo $movies->movie[0]->plot;

### exist
	isset($movies->movie);

### add

	include 'example.php';
	$movies = new SimpleXMLElement($xmlstr);

	$character = $movies->movie[0]->characters->addChild('character');
	$character->addChild('name', 'Mr. Parser');
	$character->addChild('actor', 'John Doe');

	$rating = $movies->movie[0]->addChild('key', 'val');
	$rating->addAttribute('type', 'mpaa');


### foreach
	foreach($movies as $k => $movie){
		echo $movie->title;
	}
	foreach($movies->children() as $k => $movie){
		echo $movie->title;
	}

# String

## range string

	function rangeChar($lower, $upper) {
		++$upper;
		for ($i = $lower; $i !== $upper; ++$i) {
			yield $i;
		}
	}
	foreach (rangeChar('hilojack', 'hilojacz') as $v)

	range('A', 'Z')
	array_merge(range('A', 'Z'), range('a', 'z'));

## wildcard

	bool fnmatch('*.weibo.cn', $host, FNM_CASEFOLD);//Caseless match

## csv

	str_getcsv("a,b\n\n");//忽略\n

## preg

	preg_quote('\\1'); \1 -> \\1

## split

	array str_split ( string $string [, int $split_length = 1 ] )


## DOM
Operation html string like jquery

https://code.google.com/p/phpquery/

## ini

	parse_ini_file('a.ini')
	parse_ini_string('a.ini')

	get_defined_constants($categorize = true)['user']
	parse_ini_file('a.ini') + get_defined_constants(true)['user']

### Create ini

	$arr = ['country'=> [1=>'北京\'"']];
	echo array2ini($arr);
	function array2ini($arr, $key_prefix = ''){
		$key_prefix = $key_prefix === '' ? '' : $key_prefix . '.';
		$ini = '';
		foreach($arr as $k=>$v){
			if(is_array($v)){
				$ini .= array2ini($v, "$key_prefix$k");
			}else{
				$ini .= "$key_prefix$k = \"" . str_replace('"', '\"', $v) . "\"\n";
			}
		}
		return $ini;
	}

## Double Quote

	"$str"
	"${str}"
	"{$str}"

	"$arr[0]"
	"${arr[0]}"
	"{$arr[0]}"

	"$arr['hello']" //fatal error
	"${arr['hello']}"
	"{$arr['hello']}"
	"$arr[hello]"	//notice
	"${arr[hello]}" //notice
	"{$arr[hello]}" //notice

	"$obj->key";		equal with {$obj->key}
	"{$obj->key}";
	"${obj->key}";		return value as variable name

	"{func()}";//literal
	"${func()}";//exec func(), then use the func's return value as variable name

	"${$obj->func()}" 	return value as variable name
	"${$obj->$p}" 		return value as variable name

> `\`` 反引号(grave accent)的规则同双引号一样

## heredoc & nowdoc
php 中的heredoc 与 nowdoc 和 shell 不一样，使用的是三个`<`打头，即以`<<<` 打头, 结果标记后还需要有分号`;`

	$var = <<<MM
	...
	MM;
	$var = <<<'MM'
	...
	MM;

## in string

	echo strspn("$", 'abc$');//Finds the length of the initial segment of a string within a given mask

## 大小写

	ucfirst('word word');//Word word
	ucwords('word word');//Word Word
	strtoupper();
	strtolower();

## strstr
Alias to strchr

	strstr('a@qq.com', '@qq');//@qq.com
	strstr('a@qq.com', '@qq', true);//a

## html

	nl2br
	htmlentities: encode all characters
	htmlspecialchars : encode only necessary characters
	strip_tags

Example:

	echo htmlentities('<Il était une fois un être>.');
	// Output: &lt;Il &eacute;tait une fois un &ecirc;tre&gt;.
	//                ^^^^^^^^                 ^^^^^^^

	echo htmlspecialchars('<Il était une fois un être>.');
	// Output: &lt;Il était une fois un être&gt;.
	//                ^                 ^

> http://htmlpurifier.org/ 提供了强大的html 数据清洗库

## diff
	strspn($str1, '0123456789');//求相同
	strcspn($str1, $str2);//求不同

equal:

	(ord($c) ^ 0x31) === ($c === "\x31")

## replace
strtr — Translate characters or replace substrings

	strtr($str, $arr) === str_replace(array_keys($arr), array_values($arr), $str);

	strtr($str, $from, $to);
	echo strtr('a', 'abc', 'ABC');//A

## split

	count_chars('ab',1 ); //array(97=>1, 98=>1)
	| php -r 'var_dump(array_filter(count_chars(file_get_contents("php://stdin"))));'
	| php -r 'var_dump(count_chars(file_get_contents("php://stdin"))[9]);'

## repeat

	str_repeat($str, $num);

## pad
	str_pad($str, $len, $pad, $type);

## preg regx

	int preg_match ( string $pattern , string $subject);
	array preg_grep ( string $pattern , array $input);

## format

### parse format
	// get author info and generate DocBook entry
	$auth = "24\tLewis Carroll";
	$n = sscanf($auth, "%d\t%s %s", $id, $first, $last);

	//parse rgb
	list($r, $g, $b) = sscanf('0x00ccff', '0x%2x%2x%2x');


### sprintf

	sprintf($format, $var1, $var2);
	printf($format, $var1, $var2);

## 统计字符

### substr_count

	substr_count

### count_chars
	count_chars($str, $mode=0)

Depending on mode count_chars returns one of following:

- 0 - an array with the byte-value as key and the frequency of every byte as value;

### strword_count
	str_word_count

## nowdoc & heredoc
heredoc 会解释变量

	echo <<<EOT
	My name is "$name". I am printing some $foo->foo.
	Now, I am printing some {$foo->bar[1]}.
	This should print a capital 'A': \x41
	EOT;

而nowdoc 不会解释变量(原样输出）

	echo <<<'EOT'
	My name is "$name". I am printing some $foo->foo.
	Now, I am printing some {$foo->bar[1]}.
	This should print a capital 'A': \x41
	EOT;

# i18n(gettext)

	setlocale(LC_ALL, 'en_US');

	//locate language file
	bindtextdomain('messages', '/usr/local/locale/')
		//locale/en_US/LC_MESSAGES
	//locate messages in en_US(MESSAGES)
	textdomain('messages');

	//create text file: messages.po
	$ xgettext -n *.php
	//create binary file: messages.mo
	$ msgfmt messages.po

	//translate
	gettext('hello')

# pack & unpack

	string pack ( string $format [, mixed $args [, mixed $... ]] )

Note the distinction between signed and unsigned values only affect unpack function, where as function pack() gives the same result for signed and unsigned format codes.

	$format
	a	NUL-padded string
	A	SPACE-padded string

	bin2hex(pack('a3', '1'));//"310000"
	bin2hex(pack('A3', '1'));//"312020"

	h	Hex string, low nibble first
	H	Hex string, high nibble first

	bin2hex(pack('H', 0x4));	//4 -> 4 -> 0x40
	bin2hex(pack('H', 0x12));	//18 -> 1 -> 0x10
	bin2hex(pack('H', 0x15));	//21 -> 2 -> 0x21
	bin2hex(pack('H2', 0x15));	//21 -> 21 -> 0x21
	bin2hex(pack("H2", "21")); //"21" ->21 -> "21"
	bin2hex(pack("H*", "123")); //"123"-> 12 3 -> "1230"

	unpack("H", "\x12\x3");//["1"]
	unpack("H2", "\x12\x3");//["12"]
	unpack("H3", "\x12\x3");//["120"]
	unpack("H*", "\x12\x3");//["1203"]
	unpack("h", "\x12\x3");//["2"]
	unpack("h2", "\x12\x3");//["21"]
	unpack("h3", "\x12\x3");//["213"]
	unpack("h*", "\x12\x3");//["2130"]

	bin2hex(pack('h', 0x15));//21 -> 2 -> 0x02
	bin2hex(pack('h', 0x12));//18 -> 1 -> 0x01
	bin2hex(pack('h2', 0x15));//21 -> 21 -> 0x12
	bin2hex(pack('h', "\x31"));//"1" -> 1 -> "01"
	bin2hex(pack('h2', "123"));//"123" -> 12 -> "21"
	bin2hex(pack('h*', "123"));//"123" -> 12 3-> "2103"

	c	signed char
	C	unsigned char
	bin2hex(pack('c', 10));//10 -> 0x0a
	unpack('c', "\x0a"));//0x0a -> [10]
	unpack('c', "\xff"));//0x0a -> [-1]

	s	signed short (always 16 bit, machine byte order)
	S	unsigned short (always 16 bit, machine byte order)
	bin2hex(pack('s', 10));//10 -> 0x0a00
	bin2hex(pack('s', 0x1234));//0x1234 -> 0x3412
	bin2hex(pack('s', 0xffff));//0xffff -> 0xffff

	dechex(unpack('s', "\x34\x12")[1]);	//0x3412 -> 0x1234
	unpack('s', "\xff\xff")[1];	//0xffff -> -1
	unpack('S', "\xff\xff")[1];	//0xffff -> 65536

	n	unsigned short (always 16 bit, big endian byte order)
	v	unsigned short (always 16 bit, little endian byte order)
	bin2hex(pack('n', 0x1234));//0x1234 -> 0x1234
	bin2hex(pack('v', 0x1234));//0x1234 -> 0x3412

	i	signed integer (machine dependent size and byte order)
	I	unsigned integer (machine dependent size and byte order)
	l	signed long (always 32 bit, machine byte order)
	L	unsigned long (always 32 bit, machine byte order)
	bin2hex(pack('i', 0x1234));//0x1234 -> 0x34120000 my 32bit
	bin2hex(pack('l', 0x1234));//0x1234 -> 0x3412

	N	unsigned long (always 32 bit, big endian byte order)
	V	unsigned long (always 32 bit, little endian byte order)
	bin2hex(pack('N', 0x1234));//0x1234 -> 0x00001234
	bin2hex(pack('V', 0x1234));//0x1234 -> 0x34120000

	new in php 5.6
	q	signed long long (always 64 bit, machine byte order)
	Q	unsigned long long (always 64 bit, machine byte order)
	J	unsigned long long (always 64 bit, big endian byte order)
	P	unsigned long long (always 64 bit, little endian byte order)
	f	float (machine dependent size and representation)
	d	double (machine dependent size and representation)
	x	NUL byte
	X	Back up one byte
	Z	NUL-padded string (new in PHP 5.5)
	@	NUL-fill to absolute position

# network

## curl
Refer to [php-curl](https://github.com/hilojack/php-lib)

## headers

### getheader

	getallheaders();

## dns

	checkdnsrr($domain) && print "domain exists!" ;
	//获取dns信息
	print_r(dns_get_record('baidu.com'));
	//获取mx信息(mail exchange 记录)
	getmxrr('baidu.com', $info); print_r($info);

## service

	//get port
	echo getservbyname($service = 'http', $protocol = 'tcp');//service 必须是/etc/services中
	//get service name
	echo getservbyport($port = 80, 'tcp');

## socket

	$http = fsockopen('baidu.com', 80[, $errno, $err, $timeout]);
	fputs($http, $header = "GET /1.1\r\nConnection: Close\r\n\r\n");
	while(!feof($http)) $res = fgets($http, 1024);

## mail

	mail($to, $subject, $message[, $header = "From: test@hilojack.com\r\n"]);
	cat php.ini
		sendmail_from = string
		sendmail_path = String 默认为系统sendmail的路径
		smtp_port = 25(default)

## ip

	ip2long('0.0.1.1');//257
	long2ip

# Time

## timestamp

### get

	time(viod);//get current timestamp
	getlastmod(void);// 返回当前php文件的modifier timestamp

### mktime

	mktime($hours,$minutes,$secs,$month, $day, $year); //make timestamp
	strtotime('-2 days');
	strtotime('Next Monday');

### strtotime
last week:

	today is 10151106 Friday
	echo date("Ymd", strtotime("last monday"));
		20151102
	echo date("Ymd", strtotime("last week"));
		20151026

## locale

	setlocal(LC_ALL, 'zh_CN.utf8');

## date

	date('Y-m-d')
	echo date('Y-m-d', strtotime('+2 days'));

## DateTime
> http://wulijun.github.io/php-the-right-way/#date_and_time

	$date = new DateTime;
	$date = DateTime::createFromFormat('d.m.Y','13.7.1999');

	echo $date->format('Y-m-d H:i:s');

计算时间一般需要DateInterval 类，.add .sub 都使用其作为参数, .diff 返回的值就是DateInterval：
Refer to [DateInterval](http://jp2.php.net/manual/zh/dateinterval.construct.php)

	$start = new DateTime();
	// create a copy of $start and add one month and 6 days
	$end = clone $start;
	$end->add(new \DateInterval('P1M6D'));

	$diff = $end->diff($start);
	echo "Difference: " . $diff->format('%m month, %d days (total: %a days)') . "\n";

# user

	get_current_user() Gets PHP script owner's name
	getmyuid() - Gets PHP script owner's UID
	getmygid() - Get PHP script owner's GID The user who *owns the file* containing the current script , not the user who running
	getmypid() - Gets PHP's process ID
	getmyinode() - Gets the inode of the current script
	getlastmod() - Gets time of last page modification

Get user php is running as:

	echo exec('whoami');

# Disk
	filesize
	float disk_free_space(str $dir); //返回目录所在分区剩余大小
	float disk_total_space(str $dir); //返回目录所在分区大小

# Memory

	memory_get_usage($real_usage = false); //false 返回emalloc分配的内存; true:返回操作OS分配的内存:real_size
	memory_get_peak_usage($real_usage = false); //内存峰值

# Cpu

	$dat=getrusage();
	foreach ($dat as $key => $val) {
		$usertime= $dat['ru_utime.tv_usec'];
		$systemtime= $dat['ru_stime.tv_usec'];
		$finalresultcpu= ($systemtime + $usertime);
	}

# mysqli

	$mysqli = new mysqli("127.0.0.1", "", "", "test", 3306);
	if ($mysqli->connect_errno) {
		echo "Failed to connect to MySQL: (" . $mysqli->connect_errno . ") " . $mysqli->connect_error;
	}else{
		echo $mysqli->host_info . "\n";
	}
	$res = $mysqli->query("SELECT * FROM ahui limit 10");
	while ($row = $res->fetch_assoc()) {
		foreach($row as $k => $v){
			echo "$k=$v,\t";
		}
		echo "\n";
	}

# Shell

## 转义

	str escapeshellarg($arg); //将args转义为普通字符串: 比如`abc`, 转义成`'abc'`
	str escapeshellcmd($cmd); //转义cmd中的特殊字符：| ; & > \x0a ....

## 执行

	passthru($cmd); //直接打印输出
	$output = shell_exec($cmd); // 相当于反引号 $output = `$cmd`
	$output = system($cmd[, $errno]); //也会直接打印输出
	$lastline = exec($cmd[, $output, [, $errno]])

shell_exec 等价于反引号:

	echo `ls`;
	echo `'ls'`;
	echo `"ls"`;
	echo `$cmd`;

## pipe

	<?php
	$descriptorspec = array(
		0 => array("pipe", "r"),  // stdin is a pipe that the child will read from
		1 => array("pipe", "w"),  // stdout is a pipe that the child will write to
		//2 => array("file", "/tmp/error-output.txt", "a") // stderr is a file to write to
		2 => array("file", "a.txt", "a") // stderr is a file to write to
	);
	$process = proc_open('php', $descriptorspec, $pipes, $cwd =null, $env =['env1' => 'cookie']);
	if (is_resource($process)) {
		fwrite($pipes[0], '<?php print_r($_ENV); ?>');
		fclose($pipes[0]);

		echo stream_get_contents($pipes[1]);
		fclose($pipes[1]);

		// It is important that you close any pipes before calling
		// proc_close in order to avoid a deadlock
		$return_value = proc_close($process);

		echo "command returned $return_value\n";
	}


# shutdown
register_shutdown_function($callback);

## fastcgi
> http://stackoverflow.com/questions/4806637/continue-processing-after-closing-connection

If running under fastcgi you can use the very nifty:

	fastcgi_finish_request();

This allows for time consuming tasks to be performed without leaving the connection to the client open.

> http://www.laruence.com/2008/04/16/98.html

## client abort
`php.ini`:

	ignore_user_abort False;" by default. If changed to TRUE scripts will not be terminated after a client has aborted their connection.

In php running:

	ignore_user_abort();

使用nginx的童鞋注意:

	fastcgi_ignore_client_abort on; // 客户端主动断掉连接之后，Nginx 会等待后端处理完(或者超时);


# php package manager
目前主要有两个PHP包管理系统：Composer和PEAR

- 管理单个项目的依赖时使用Composer
- 管理整个系统的PHP依赖时使用PEAR

## pear

	curl http://pear.php.net/go-pear | php

pear(Php Extension and Aplication Repository) 好像没有composer 流行

	pear list
	pear help
	pear instal package

# Interface

	interface a {
		public function foo();//相当于 abstract public function foo();
	}

	interface b extends a {
		public function baz(Baz $baz, array $c = []);//相于abstract 原型
	}

	class c implements b {
		public function foo() { }
		abstract public function foo();//false, 只有abstract class 才允许abstract function
		protected function foo() {}; //Access level must be public 这是父级决定的

		public function baz(Baz $baz, $c) { }			//error
		public function baz(Baz $baz, array $c) { }		//error
		public function baz(Baz $baz, array $c = []) { }//ok
		public function baz(Baz $baz, array $c = [1]) { }//ok
	}
	new c;
	abstract class c implements b {
		abstract public function foo();//ok
	}

> 子类不可改变父类的 public/protected/privated 的access level, 也不可以改变参数的数据类型(子类型也不行)

> extends/implements 所继承的类或接口，必须先定义后使用. 它本身的定义也需要先定义再使用
否则编译器会报错

# Extension
http://php.net/manual/en/refs.basic.other.php

## Tokenizer
http://php.net/manual/en/book.tokenizer.php

## APCu & Memcached (对象缓存)
APCu 在单机性能上比 Memacached 高，但是它不能跨进程，更不到跨机器

> Note that when running PHP as a (Fast-)CGI application inside your webserver, every PHP process will have its own cache, i.e. APCu data is not shared between your worker processes. In these cases, you might want to consider using memcached instead, as it’s not tied to the PHP processes.

	apc_fetch('key');
	apc_add('key', $data);

在php5.5 之前APC 作为扩展时，即提供opcode 字节码缓存，又提供对象缓存。
但是php5.5 之后集成了字节码缓存组件OPcache, APC 的对象缓存部分被单独提出来了，叫做APCu

## SPL(Standard PHP Library)
标准库是php 自带的extension `php -m | grep SPL`，它提供了：常用的数据结构如栈，队列，堆, 数组, 以及这些数据结构的迭代器。如果需要还可自己扩展这些SPL

Refer to [](/p/php-spl)

## Taint
It is used for detecting XSS codes(tainted string). And also can be used to spot sql injection vulnerabilities, and shell inject, etc.

## Other Extension
V8js,Yaf,
GeoIP — Geo IP Location
Yaml — YAML Data Serialization
