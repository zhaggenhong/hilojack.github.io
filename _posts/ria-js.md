---
layout: page
title:	js notes
category: blog
description:
---
# Preface
本文参考: [](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript)

# todo
https://leanpub.com/exploring-es6/read
https://es6.ruanyifeng.com/
javascript 一站式 http://www.liaoxuefeng.com/wiki/001434446689867b27157e896e74d51a89c25cc8b43bdb3000

# "use strict";
use strict参考：阮一峰 http://www.ruanyifeng.com/blog/2013/01/javascript_strict_mode.html

必须写在执行语句或者函数体内的第一行

1. 变量必须有var 声明
2. 不允许用with
3. 正常模式下eval 内的变量作用域取决于所在的作用域。strict 模式下，eval 本身是一个作用域
4. function 下的this 是undefined, 而new 生成的obj 内的this 指向自己

## 禁止删除变量
严格模式下无法删除变量。只有configurable设置为true的对象属性，才能被删除。

	"use strict";
　　var x;
　　delete x; // 语法错误
　　var o = Object.create(null, {'x': {
　　　　　　value: 1,
　　　　　　configurable: true
　　}});
　　delete o.x; // 删除成功

## 显式报错

### 对只读变量写入
正常模式下，对一个对象的只读属性进行赋值，不会报错，只会默默地失败。严格模式下，将报错。

	"use strict";
　　var o = {};
　　Object.defineProperty(o, "v", { value: 1, writable: false });
　　o.v = 2; // 报错

### 重名报错

	"use strict";
　　var o = {
　　　　p: 1,
　　　　p: 2
　　}; // 语法错误

# this scope
定义函数时的的this: 指向window
执行函数时的this: 函数所在object

## 被传递的函数
其里面的this 均表示window.(jquery event call 改变了这个this)

无论它是匿名:

	str = new String('xx');
	str.func = function(f){
		f();
	}
	str.anonymousFunc = function(){
		this.func(function(){console.log(this)});

	}
	str.anonymousFunc()

还是对象函数, 或者全局函数

	str = new String('xx');
	str.func = function(f){
		f();
	}
	str.objFunc = function(){
		console.log(this);
	}
	str.func(str.objFunc)

## 被调用的函数
this 是指向定义处的this

## call/apply
用call/apply改变方法的this指向它所传的对象, 详见继承

	func.call(obj)

    function foo() { console.log(arguments); }

    // 1. directly
    foo(1, 2, 3);

    // 2. trough Function.call()
    foo.call(this, 1, 2, 3);

    // 3. trough Function.apply()
    var args = [1, 2, 3];
    foo.apply(this, args);

# var scope
匿名函数中的scope 是定义所在的scope:
global -> caller -> callback(anonymous)

	v="I'm global";
	function call(callback){
		var v="I'm call";
		callback();
	}
	function caller(){
		var v="I'm caller";
		call(function(){console.log(v)});
	}
	caller();//I'm caller

普通函数中的scope 是定义所在的scope:

	v="I'm global";
	function call(callback){
		console.log(v);
	}
	function caller(){
		var v="I'm caller";
		call();
	}
	caller();//I'm global

# Condition Expression

	switch(n) { case 1: code break;}
	带类型匹配

# break

	break [label];

	function foo () {
		dance:
		for(var k = 0; k < 4; k++){
			for(var m = 0; m < 4; m++){
				if(m == 2){
					break dance;
				}
			}
		}
	}

# Variable
强制类型转换:

	Boolean(value) - 把给定的值转换成 Boolean 型；
	Number(value) - 把给定的值转换成数字（可以是整数或浮点数）；
	String(value) - 把给定的值转换成字符串；

## type

	Object.prototype.toString.call(1)
	"[object Number]"
	"[object Arguments]"

    var args = [];
    Array.prototype.push.apply( args, arguments );
    [].shift.call(arguments)

to string:

	String(value)
	value + ""
	undefined + ""
	'0'+1
		'01'

## Compare
[js Boolean比较问题研究](http://www.joglex.com/?p=189)

to number:

	[]==[];//false 因为它们指向不同的对象. 这跟php是不一样的.
	[]==0; //true Number({})的值为NaN，而Number([])的值为0 Number(null)为0 Number(undefined) = NaN
	{}==0; //false

to Boolean:

	null==undefined; //true
	0==''==false;//true

	![];//false, 对象取反都为false *******
	!null;!undefined;!'';!0; //true


## Array

	new Array(1,2,3);
	arr = [1,2,'d'];
	brr[0]=11;
	arr;//[11,2,'d']; //因为数组赋值使用的是对象引用

	.length
	.concat(arr2)
	.join([separator]); //implode
	.pop() ;//返回item
	.push(item1, itme2, ...);//return length
	.reverse()
	.shift();//左移(移出) .unshift(item1,item2);//右移(移入)

	.slice(start, howmany); //支持负数
	.splice(start, howmany[, newValue]);//删除
        a=[1,2,3,4,5]
            [1, 2, 3, 4, 5]
        a.splice(2,2,'abc')
            [3, 4]
        a;
            [1, 2, "abc", 5]
	.sort([funcSort]);

	转换:

		arr.toString();arr.valueOf();
		arr.join(',');//

	栈方法:

		arr.push('new item');
		arr.pop();

	队列:

		arr.push('new');
		arr.shift();//LIFO 后进先出

	重排方法(改变arr本身):

		arr.reverse();//倒序
		arr.sort();//从小到大
		arr.sort(function(a,b){return a-b;});//自定义排序

	操作方法:

		[0,1].concat(2,[3,4]);//返回数组[0,1,2,3,4];
		arr.slice(start,[end]);//范围不含end本身,end可为负数,
		arr.splice(start,[length]);//返回删除范围
		arr.splice(start,length,val1,val2,....);//返回删除范围,并插入数据

	位置方法:

		['aa','bb','cc'].indexOf('aa');//0 找不到就返回-1
		['aa','bb','cc','aa'].lastIndexOf('aa');//3 找不到就返回-1

	迭代方法:

		.every(func);//每一项运行给定函数都返回true,结果才返回true. //func= function(item,index,array){}; array是对数组本身的引用
		.some(func);//只要其中一项运行指定函数时返回true,结果就返回true.
		.filter(func);//返回运行为true item组成的数组
		.map(func);//返回函数运行结果组成的数组
		.forEach(func);//只运行不返回

	归并方法:

		.reduce(func);//从第一项开始
		.reduceRight(func);//从最后一项开始
		arr.reduce(function(pre,cur,index,array){return pre+cur;})
		[1,2,3].reduce(function(pre,cur,index,array_self){return pre+cur;})
			array_self 作为数组是按引用传值的
			prev = prev+ curr = 1+2 = 3;
			prev = prev+ curr = 3+3 = 6;

	Array.prototype.inArray = function(needle) {
        var length = this.length;
        for(var i = 0; i < length; i++) {
            if(this[i] == needle) return true;
        }
        return false;
    }

## Date

	$d = new Date("October 13, 1975 11:13:00");
	$d = new Date(miliseconds);

	//date & time
	Date(); 		"Mon Apr 28 2014 23:25:02 GMT+0800 (CST)"
	.toString(); 	"Mon Apr 28 2014 23:25:02 GMT+0800 (CST)"
	.toUTCString();	"Mon, 28 Apr 2014 15:25:02 GMT" //UTC & GMT 是一样的(除了精度上有区别)
	.toGMTString();	"Mon, 28 Apr 2014 15:25:02 GMT"
	.toLocaleString(); "4/28/2014 11:25:02 PM"

	//date part
	.getDate(); //1~31 getUTCDate()
	.getMonth(); //0~11 getUTCMonth()
	.getFullYear(); //四位数字	getUTCFullYear()
	//day
	.getDay(); //一周中的某天0~6(0是星期天)

	//time part
	.getHours()	返回 Date 对象的小时 (0 ~ 23)。 getUTCHours()
	.getMinutes()	返回 Date 对象的分钟 (0 ~ 59)。	getUTCMinutes()
	.getSeconds()	返回 Date 对象的秒数 (0 ~ 59)。 getUTCSeconds()
	.getMilliseconds()	返回 Date 对象的毫秒(0 ~ 999)。getUTCMilliseconds()

	//unixstimestamp in miliseconds
	.valueOf()
	.getTime()	返回 1970 年 1 月 1 日至今的毫秒数。
	Date.parse("Jul 8, 2005 0:0:32"); //返回指定时间的毫秒数
	.UTC()	根据世界时返回 1970 年 1 月 1 日 到指定日期的毫秒数。

	//unixstimestamp in seconds
	new Date().getTime()/1000 | 0
	+new Date/1000 | 0

short:

	Date.prototype.formatMMDDYYYY = function(){
		return (this.getMonth() + 1) +
				"/" +  this.getDate() +
				"/" +  this.getFullYear();
	}
	Date.prototype.format = function(d){
		var d = this;
		return d.getFullYear()+'-' + (d.getMonth()+1)+'-' + d.getDate()+' ' + d.getHours()+':' + d.getMinutes()+':' + d.getSeconds()+''
	}

### compare

	d1 > d2

### set

	//day
	.setDate()	设置 Date 对象中月的某一天 (1 ~ 31)。

	//Month
	.setMonth()	设置 Date 对象中月份 (0 ~ 11)。

	//Year
	.setFullYear()	设置 Date 对象中的年份（四位数字）。 setYear()	使用 setFullYear() 方法代替。

	//Hours & Minutes & Seconds
	.setHours()	设置 Date 对象中的小时 (0 ~ 23)。
	.setMinutes()	设置 Date 对象中的分钟 (0 ~ 59)。
	.setSeconds()	设置 Date 对象中的秒钟 (0 ~ 59)。
	.setMilliseconds()	设置 Date 对象中的毫秒 (0 ~ 999)。

	//add time
	d.setSeconds(d.getSeconds() + 10);
	Date.prototype.add= function(seconds){
		this.setTime(this.getTime() + (seconds * 1000));
		return this;
	}

	//Time
	.setTime(millisec)	以毫秒设置 Date 对象。d.setTime(77771564221)
	new Date(millisec)	以毫秒设置 Date 对象. new Date(77771564221)

str to time

	new Date("October 13, 1975 11:13:00");
	new Date("2016-01-01 11:13:00");
	(new Date("2016-1-1")).getDay();

## String

	'\x31'
	"\x31"

### function

	.charAt(pos) str[pos]
	.charCodeAt(pos) //返回ascii十进制表示
	.fromCharCode(97)
	#搜索
	.indexOf(sub_string)
	.lastIndexOf(sub_str)
	#compare
	.localeCompare(str) //return 1 0 -1
	#截取
	.substr(start,[length])//start可为负
	.slice(start, [end]) //start, end可为负
	#case
	.toLowerCase() / .toUpperCase()

### .trim()

	' a '.trim()

### unicode

	'好' === '\u597D' // true
	'好' === '\u{597D}' // true

### for

	for(var i=0; i < str.length; i++ ){
		str[i];
	}

### base64

	btoa(str)
	atob(str)

### split
支持regexp

	stringObject.split(separator,[maxSize]);
	'1,2,3,4,5'.split(',', 3);//[1,2,3]
	'1,2,3,4,5'.split(/,/);//[1,2,3,4,5]

### font

	str.link(url)
	str.bold()
	str.sup() 上标
	str.sub() 下标 //"<sub>str</sub>"
	str.small() 小字号
	str.big()
	str.fontcolor('red') "<font color="red">s</font>"
	's'.fontsize('1px')

### Match

	//return 存放匹配结果的数组
	stringObject.match(searchvalue)

	stringObject.match(regexp)
	matches = ("1 2 3 ".match(/\d+/g); //[1,2,3]
	"1 2 3".match(/(\d) s/g); //return null

	//如果regexp没有g, 则会匹配子模式
	matches = "first 1".match(/(\w+) 1/);//如果没有g, 则返回包括子表达式  ["first 1", "first"]
	matches.index //0 相当于indexOf返回的位置

> 如果需要同时支持regExp global 及 子表达式, 请使用RegExp.prototype.exec

### search
返回字符位置, 不支持regexp global; 这个像indexOf()

	str.search('string') equal to str.indexOf('string'); //返回字符位置
	str.search(/Hilo/i);

### replace
支持regexp global.

	str = stringObject.replace(regexp/substr,replacement)
	replacement:
		string:
			字符	替换文本
			$1、$2、...、$99	与 regexp 中的第 1 到第 99 个子表达式相匹配的文本。
			$&	与 regexp 相匹配的子串。
			$`	位于匹配子串左侧的文本。
			$'	位于匹配子串右侧的文本。
			$$	$转义
		function:
			function(mathStr, first, second, ...){return replace;}

eg:

	//reference
	'funing smoking '.replace(/(\w+)ing/g, '$1');// "fun smok "
	//func
	card = '[card]google[/card]http://g.cn'; // to "<a href="http://g.cn">google</a>"
	card.replace(/\[card\](\w+)\[\/card\]([\w:\/.]+)/,function(ori,card,url){
		return '<a href="'+url+'">'+card+'</a>';
	})

## RegExp
Create RegExp：

	/pattern/attributes
	/str/igm
	new RegExp('pattern', 'attributes');
	new RegExp("str",'igm')
	new RegExp("^str$",'igm')
	new RegExp("(^|&)str$",'igm')


	/\u1321/.test('\u1321');//true
	/\x31/u.test('\x31');//true

### zero-width 断言
正向断言，假定该位置后跟的是X

	(?<=X)	zero-width positive lookbehind
	(?=X)	zero-width positive lookahead

	(?<!X)	zero-width negative lookbehind
	(?!X)	zero-width negative lookahead

	'?_b=1&b=2'.match(/(?<=[?&])b=(\d+)/)

### test

	test	检索字符串中指定的值。返回 true 或 false。
	str=' 1 2 3';
	r=/\d/igm;
	while(r.test(str) === true){
		console.log(r.lastIndex);//2 4 6 下次要匹配的字符串起始位置
	}

### comile

	r.compile(/\d/); //改变正则表达式

### exec

	str=' 1ing 2ing 3ing';
	r=/\d(ing)/igm;
	while((match = r.exec(str)) !== null){
		console.log(match,r.lastIndex);
		//第1次输出 ["1ing", "ing", index: 1, input: " 1ing 2ing 3ing"] 5
		//第2次输出 ["2ing", "ing", index: 6, input: " 1ing 2ing 3ing"] 10
		//第3次输出 ["3ing", "ing", index: 11, input: " 1ing 2ing 3ing"] 15
	}

## Func
这里罗列的是顶层函数（全局函数）

### 转码:
常规字符: 字母 数字 下划线

	encodeURI()	把字符串编码为 URI。
		encodeURI("http://www.google.com/a file with spaces.html"); //转码所有非常规URI字符转码: '|" Ò' 等等
	encodeURIComponent()	把字符串编码为 URI 组件。(所有URI特殊字符 将被转码)
	escape()	对字符串进行编码。Don't use it, as it has been deprecated since ECMAScript v3.
	str.replace(/'/g, "\\'");//addslashes


	eval()	计算 JavaScript 字符串，并把它作为脚本代码来执行。

## Number

	数值范围:Number.MIN_VALUE~Number.MAX_VALUE (一般是5e-324~1.79769e+308)
	parseInt('0x10', [进制]) //number or NaN
	parseFloat(var) //number or NaN
	Number(undefined);//NaN
	Number(null);//0
	(10).toString(16);//'a'
	num.toFixed(2); //两位小数
	num.toExponential([小数位数]);// 科学计数法
	num.toPrecision([有效位数]); //据最有意义的形式来返回数字的预定形式或指数形式

## hex2str & str2hex

	function str2hex(str) {
		var hex = '';
		for(var i=0;i<str.length;i++) {
			hex += ('0'+str.charCodeAt(i).toString(16)).substr(-2);
		}
		return hex;
	}
	function hex2str(hex) {
		var str = '';
		for (var i = 0; i < hex.length; i += 2)
			str += String.fromCharCode(parseInt(hex.substr(i, 2), 16));
		return str;
	}

## Math

	E	返回算术常量 e，即自然对数的底数（约等于2.718）。
	LN2	返回 2 的自然对数（约等于0.693）。
	LN10	返回 10 的自然对数（约等于2.302）。
	LOG2E	返回以 2 为底的 e 的对数（约等于 1.414）。
	LOG10E	返回以 10 为底的 e 的对数（约等于0.434）。
	PI	返回圆周率（约等于3.14159）。
	SQRT1_2	返回返回 2 的平方根的倒数（约等于 0.707）。
	SQRT2	返回 2 的平方根（约等于 1.414）。

	Math.round(1.5); //2
	Math.round(1.4); //1
	Math.floor(1.5); //1
	Math.ceil(1.5);	//2

## Boolean
null == undefined 可相等.

但它们与false/true/0 都不能相等
	null==0 //return false
	undefined==0 //return false
	undefined==false //return false


# 运算符

## 一元
	delete variable
	delete obj.name

## Bits Operation 位运算

	num & num
	num | num
	~num 取反
	num ^ num	//xor
	#位移
	1<<2 # 1<<34 == 1<<2
	#有符号右移(高位1不会变)
	a=1<<31;
	a>>32 === a>>0 === a;

	#无符号右移(高位1会变成0),输出无符号数
	a
	-2147483648
	a>>>0
	2147483648
	a>>>1
	1073741824
	a>>>32
	2147483648

## 逻辑
	!var
	var && var
		逻辑 AND 运算并不一定返回 Boolean 值：
			如果一个运算数是对象，另一个是 Boolean 值，返回该对象。
				1 && obj //return obj
				0 && obj // return 0
				obj1 && obj2 //return obj2(if obj1不为空)
			如果某个运算数是 null
				1 && null //return null
				0 && null //return 0
			如果某个运算数是 NaN
				1 && NaN //return NaN
				0 && NaN //return 0
	var || var
		逻辑 OR 运算并不一定返回 Boolean 值(同上)

# 语句

## 迭代
	for(i in obj){obj[i]...} // PropertyIsEnumerable
	for(i=0;i<5;i++){...}

# Function 函数

## anonymous func
You can define a anonymous function via named function:
Example:


	//factorial
    (function(n){
    	var self = function(n){
    		//call self
			return n > 0 ? (self(n-1) * n) : 1;
    	}
    	return self;
    })()

## static variable 静态作用域
可以构造一个:

	a = function(){
		var self=a;
		self.name = 1;//static variable
	}
	a.name=1;//函数静态变量

## 闭包

	变量的作用域链.

## arguments
如果arguments[0] 存在, name 严格指向arguments[0]:
否则name 不变

	function a(name){
		arguments[0]='v11';//same as: name='v11'
		console.log(arguments, name);
	}
	a('v1','v2'); //output: ['v11', 'v2'], 'v11'

如果想让 arguments 支持数组函数:

    function f(a){
        [].shift.call(arguments)
        console.log(arguments, a);
    }
    f(1); //[],1
    f(1,2); //[2],2
    f(1,2,3); //[2,3],2
    f(1,2,3,4); //[2,3,4],2

push.apply

    var args = [];
    Array.prototype.push.apply( args, arguments );

slice:

    //注意arguments 中的object/array 还是近引用传值, string 不是
    var params = Array.prototype.slice.call(arguments);
    params.shift();

## function 对象

	new a(); 函数本身就是一个类似php 的 __constructor.
	a.length; //参数数量

# Object

## keys
list forEach

	Object.keys({a:1, b:2})
		["a", "b"]
	Object.keys(obj).forEach(function(key){
		console.log(key, obj[key])
	})

## has key value

	if('key' in myObj){ }

	function hasValue(obj, key, value) {
		return obj.hasOwnProperty(key) && obj[key] === value;
	}

## extend

	Object.prototype.extend = function(defaults) {
		for (var i in defaults) {
			if (!this[i]) {
				this[i] = defaults[i];
			}
		}
	};

## 判断原型
	obj instanceof funcname// 函数原型名

## Undefined && Null
没有值和空值

	typeof null; "object"
	typeof undefined; "undefined"

## Property 属性
.constructor(指向函数)
	对创建对象的函数的引用（指针）。对于 Object 对象，该指针指向原始的 Object() 函数。

.Prototype(指向对象原型)
	对该对象的对象原型的引用。对于所有的对象，它默认返回 Object 对象的一个实例。


.hasOwnProperty('property')
	判断对象是否有某个特定的属性

.IsPrototypeOf(object)
	判断该对象是否为另一个对象的原型。

.propertyIsEnumerable
	判断给定的属性是否可以用 for...in 语句进行枚举。

## value

	Object.prototype.hasOwnValue = function(val) {
		for(var prop in this) {
			if(this.hasOwnProperty(prop) && this[prop] === val) {
				return true;
			}
		}
		return false;
	};

### Object.defineProperty()
	obj[name] = value;
	//or
　　Object.defineProperty(obj, name, { value: value, writable: false });

Example1，在ES5 中Prototype 可以用来将定义魔法属性，可以实现类似 PHP 类的`__get`, `__set`

	Number.prototype = Object.defineProperty(
	  Number.prototype, "double", {
		get: function (){return (this + this)}
	  }
	);
	(3).double;//6
	3['double'].double;//6
	3..double;//6 第一个点被解析为小数点，第二个点被解释为操作符

对原始类型做对象操作时，js 会用原始类型的对象wrapper 把变量包装一下，然后在临时对象上操作。

	var a=3; //是数值，不是数值对象
	a.key=1; //js 数据类型的对象wrapper 会将a 包装为临时的数值对象. 相当于`(new Number(a)).key=1`
	a.key;//undefined 因为临时对象不存在了

## 创建对象

1. obj = new Object();
2. person={firstname:"John"};
3. obj = new func(param1, param2);

## 本地对象

	Object
	Function
	Array
	String
	Boolean
	Number
	Date
	RegExp
	Error
	EvalError
	RangeError
	ReferenceError
	SyntaxError
	TypeError
	URIError

## 定义对象的模式

### 工厂方式(deprecated)
工厂方式的点是:
1. 每次new factory 都会创建单独的函数
2. 太复杂

	function factory(v1, v2){
		obj = new Object();
		obj.param1 = v1;
		obj.param2 = v2;
		obj.func = function(){};
		return obj;
	}
	obj = factory(1,2)

### 构造方式(deprecated)
只避免了factory 的第二个缺点: 复杂性

1. 每次都会生成新的function.

	function construction(v1,v2){
		this.p1 = v1;
		this.p2 = v2;
		this.func = function(){}; //为避免重复的func, 可在外部定义
	}
	obj = new constructor(1,2)

### 原型方式(deprecated)
没有工厂方法的缺点，但产生了新的缺点：

- 不能传参数

	function Car() {
	}
	Car.prototype.color = "blue";
	Car.prototype.showColor = function() {
	  alert(this.color);
	};
	(new Car) instanceof Car;//true;
	(new Car) instanceof Object;//true;
	(new Car) instanceof Number;//false;

### 静态的构造原型混合
- 构造: 放私有的属性
- 原型: 放公共的属性

> 批评静态构造函数原型混合方式的人认为，在构造函数内部找属性，在其外部找方法的做法不合逻辑: 所以就有一种动态 构造/原型混合

#### 动态构造原型混合

	function Car(color){
		//private
		this.color=color;

		//public(prototype)
		//if (typeof Car._initialized === "undefined") {
		if (Car._initialized === undefined) {
			var self = Car;
			self._initialized = true;//非prototype 的_initialized 不会被继承，但是它相当于Car 内的静态变量
			self.prototype.showColor = function() {
			  console.log(this.color);
			};
		}
	  }
	}

## Extends 继承

### 对象冒充
ClassB 继承 ClassA

	function ClassA(color){
		this.color = color;
	}
	function ClassB(color, num){
		this.method = ClassA; //ClassB就冒充了ClassA中的this
		this.method(color)
		delete this.method;

		this.method = ClassA1; //ClassB就冒充了ClassA1中的this(可以多重继承的)
		this.method(num);
		delete this.method;

		this.color = value; //Notice; 会覆盖前面的属性. 请确保属性名不冲突
	}
	new ClassB('red', 5);

### Call()

	function sayColor(sPrefix,sSuffix) {
		alert(sPrefix + this.color + sSuffix);
	};

	var obj = new Object();
	obj.color = "blue";

	sayColor.call(obj, "The color is ", "a very nice color indeed.");// saycolor中的this会指向obj, obj对象就冒充了saycolor.

#### 用call 实现继承

	function ClassB(sColor, sName) {
		//this.newMethod = ClassA;
		//this.newMethod(color);
		//delete this.newMethod;
		ClassA.call(this, sColor);//执行时，this冒充了ClassA

		this.name = sName;
		this.sayName = function () {
			alert(this.name);
		};
	}

### apply 方法冒充继承
apply与call方法类似, 除了参数调用形式不一样.

	function sayColor(sPrefix,sSuffix) {
		alert(sPrefix + this.color + sSuffix);
	};

	var obj = new Object();
	obj.color = "blue";

	sayColor.apply(obj, new Array("The color is ", "a very nice color indeed."));

#### apply 实现继承.

	function ClassB(sColor, sName) {
		//this.newMethod = ClassA;
		//this.newMethod(color);
		//delete this.newMethod;
		ClassA.apply(this, arguments);//arguments === new Array(sColor);

		this.name = sName;
		this.sayName = function () {
			alert(this.name);
		};
	}

还有一个bind 方法，但函数使用bind 时，函数不会执行 var get = request.bind(this, 'GET', arg2, ... ); request 就不会执行。
它主要用于定制一个新的 requset 的函数，并默认函数的this 及定制参数:
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind


### 原型链（prototype chaining）
用prototype 对象去继承.
缺点: 不能控制被继承类的传参(一般都不会传参数)

	function ClassA() {
	}

	ClassA.prototype.color = "blue";
	ClassA.prototype.sayColor = function () {
		alert(this.color);
	};

	function ClassB() {
	}

	ClassB.prototype = new ClassA(); //继承了啦.

### 混合方式
对象冒充: 不能冒充静态成员(prototype public)
原型链: 因为prototype 是公共的, 所以传argument(private)就不合适了.故产生了apply/call + 原型的混合方式.

	function ClassA(color) {
		this.color = color;
		if(ClassA.init === undefined){
			ClassA.init = true;
			ClassA.prototype.sayColor = function () {
				console.log(this.color);
			};
		}
	}

	function ClassB(sColor, sName) {
		var self = ClassB;
		if( self.init === undefined){
			self.init = true;
			//self.prototype = new ClassA(); //1. 原型链冒充 原类的静态成员(prototype). 最好别传new ClassA(sColor),因为sColor应该是每个对象私有的.
			self.prototype = Object.create(ClassA.prototype)
		}
		ClassA.call(this, sColor);// 2. 再冒充ClasA对象.
		this.name = sName;
	}

用Call 继承属性方法初始化，用`Object.create` 继承公共属性与方法
再来一个例子：

	//Shape - superclass
	function Shape() {
	  this.x = 0;
	  this.y = 0;
	}

	Shape.prototype.move = function(x, y) {
		this.x += x;
		this.y += y;
		console.info("Shape moved.");
	};

	// Rectangle - subclass
	function Rectangle() {
	  Shape.call(this); //call super constructor.
	}

	Rectangle.prototype = Object.create(Shape.prototype);

	var rect = new Rectangle();

	rect instanceof Rectangle //true.
	rect instanceof Shape //true.

	rect.move(1, 1); //Outputs, "Shape moved."

# profile
- [js profile]

[js profile]: http://kb.cnblogs.com/page/501177/
