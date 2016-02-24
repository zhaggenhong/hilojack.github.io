---
layout: page
title:
category: blog
description:
---
# Preface


# Data Type(数据类型)

	NULL means you do not have to provide a value for the field... default to null
	NOT NULL means you must provide a value for the fields. 但很多情况下，插入数据时默认会给一个空值(0, 或者空字符串).

## calc time

### diff time

	> SELECT UNIX_TIMESTAMP('2010-11-29 13:16:55') - UNIX_TIMESTAMP('2010-11-29 13:13:55') as output
	180
	> SELECT ABS(UNIX_TIMESTAMP(t.datetime_col1) - UNIX_TIMESTAMP(t.datetime_col2)) as out

better solution:

	SELECT TIMESTAMPDIFF(SECOND, '2010-11-29 13:13:55', '2010-11-29 13:16:55')

### sec_to_time

	select SEC_TO_TIME(103600000)
	| 838:59:59              |

	select TIME_FORMAT(SEC_TO_TIME(3600),'%H:%m');
	| 01:00                                  |


## time

### cast & convert

	select	cast('2016-03-01 01:01:02' as date) |
	+-------------------------------------+
	| 2016-03-01
	group by year('2016-03-01 01:01:02' as date) |
	| 2016
	select	month('2016-03-01 01:01:02' as date) |
	| 3
	+---------------------------------------------+
	| DATE_FORMAT('2016-03-01 01:01:02', '%Y/%m') |
	+---------------------------------------------+
	| 2016/03                                     |

### Compare
time format:

	'20140102101213'
	20140102101213
	'2014-01-01 01:02:03'
	'01:02:03'

Compare time

	select * from table where settime > '2014-01-01 01:02:03' limit 1;
	select * from table where settime > '2014-01-01' limit 1;
	select * from table where settime > 20140102101213 limit 1;
	select * from table where settime > 20140102 limit 1;

### date & time

对于所有日期和时间格式，mysql 支持任何非字母数字作间隔的日期，比如`2010!08!10`

	DATE; //支持20101012 , 2010-8-10等
	TIME; //10:11:12
	DATETIME; //支持20100102 10:11:12, 20100102101112 等

	TIMESTAMP [DEFAULT] [ON UPDATE]; // 2015-10-12 21:52:41

### TIMESTAMP
TIMESTAMP 比较特殊，`默认`的INSERT 或者UPDATE 会触发时间更新为当前的时间(它其实不是时间戳，而是DATA+TIME 字符串):

	TIMESTAMP;
	TIMESTAMP not null;

设置了default 后，仅当insert 是才更新时间为default 对应的值:

	TIMESTAMP null; //insert 时, 会被更新为NULL
	TIMESTAMP DEFAULT CURRENT_TIMESTAMP; //insert 时, 会被更新为当前的时间
	TIMESTAMP default 20140102101213; //insert 时间会被更新为默认的值"2014-01-02 10:12:13"
	TIMESTAMP default "20140102101213"; //insert 时间会被更新为默认的值"2014-01-02 10:12:13"
	TIMESTAMP default "2014-01-02 10:12:13"; //insert 时间会被更新为默认的值"2014-01-02 10:12:13"

除非增加了on update, update 时将时间更新，insert 时默认时间为0

	d TIMESTAMP default CURRENT_TIMESTAMP on update CURRENT_TIMESTAMP;//insert 时默认时间为当前的时间
	d TIMESTAMP on update CURRENT_TIMESTAMP;//insert 时默认时间为0

#### UNIX_TIMESTAMP

	select UNIX_TIMESTAMP('2013-08-05 18:19:03');
		1375697943
	select STR_TO_DATE('2013 08 05 18:19:03', '%Y %m %d %H:%i:%s');
		2013-08-05 18:19:03

### YEAR

	YEAR [(2|4)]
		1~69 , as 2001~2069
		"00"~"69", as 2000~2069
		70~99 , as 1970~1999
		"70"~"99" , as 1970~1999
		1901~2155

## Number

	2e30
	2+3*3
	10%3

What the exactly number if value over range?

> If number is above the range, the value mysql store will be the max value.
> If number is below the range, the value mysql store will be the min value.

What does the number in parenthesis mean?

> int(2) will generate an INT with minimum display width of 2. It's up to mysql client.
In most clients, if a colume specified with `INT(2) ZEROFILL`, the number 6 will be displayed as '06'.

### function

	FLOOR(RAND() * 401) + 100

### INT

	### TINYINT
	Their signed value range is (-128,127) , and unsigned range (0,255)。

	#### BOOL & BOOLEAN
	They are TINYINT(1) alias

	### SMALLINT [(M)] [UNSIGNED] [ZEROFILL]
	Their unsigned range is (0,2^16-1)。

	### MEDIUMINT [(M)] [UNSIGNED] [ZEROFILL]
	Their unsigned range is (0,2^24-1)。

	### INT [(M)] [UNSIGNED] [ZEROFILL]
	Their unsigned range is (0,2^32-1)。

	### BIGINT [(M)] [UNSIGNED] [ZEROFILL]
	Their unsigned range is (0,2^64-1)。


### Float

#### DECIMAL ([M,D]) [UNSIGNED] [ZEROFILL]
以字符串存储浮点数。

	DECIMAL(4,1);//只能存储4位字符，小数部分1位(不含小数点和负号). 即-999.9~ 999.9 或 0.0 ~ 999.9
	DECIMAL; //Same as DECIMAL(10,0)

####  FLOAT([M,D]) [UNSIGNED] [ZEROFILL]

	FLOAT(4,1);//Same as DECIMAL(4,1). Storage as string

	FLOAT;
		//参照c 语言的IEEE 754 double 32位存储 signed: +/- 10^38, unsigned: 0~10^38

####  DOUBLE([M,D]) [UNSIGNED] [ZEROFILL]

	DOUBLE(4,1);//Same as DECIMAL(4,1). Storage as string

	DOUBLE;
		//参照c 语言的IEEE 754 double 64位存储 signed: +/- 10^308, unsigned: 0~10^308

## String
VARCHAR 不会存储尾部空白\0，而从5.0.3 开始出于兼容性考虑，会跟CHAR一样存储尾部空白。

	update talbeName set c='str' where id = 1

	"abc\n123"

### Function
> https://dev.mysql.com/doc/refman/5.0/en/string-functions.html

	SELECT CONCAT('My', 'S', 'QL');

#### length

	select length('国'); //1
	select length("中\');

### CHAR
Length 不是字节数，而是字符数

	CHAR(Length) [BINARY | ASCII | UNICODE ]

		Length
			The length can be any value from 0 to 255.
			If Length is 0, Char can only be NULL or ""
			If length is bigger than 255, it will be store as TEXT TYPE
		BINARY
			排序时区分大小写(默认不区分)
		ASCII
			使用Latin1 这种万能字符集(默认?)
		UNICODE
			使用ucs2 字符集(又字节?)

### VARCHAR

	VARCHAR(Length) [BINARY ]
		Length
			The length can be any value from 0 to 65536.
			If Length is 0, Char can only be NULL or ""
			If length is bigger than 255, it will be store as TEXT TYPE
		BINARY
			排序时区分大小写(默认不区分)

### Other

	LONGBLOB
		支持2^32-1个字符, 最大的二进制存储
	LONGTEXT
		支持2^32-1个字符, 最大的文本存储(不能存储\0)
	MEDIUMBLOB/MEDIUMTEXT
		2^24-1
	BLOB/TEXT
		2^16-1 = 65535
	TINYBLOB/TINYTEXT
		2^8-1 = 255

### Limit string set
String set

#### ENUM

	ENUM("str1",...., "str65535")
	//如果声明了NOT NULL, 则默认第一个，否则默认NULL

#### SET
SET 可以指定预定值中的一个或者多个值


	set("str1","str2", ....)
	insert table values('str1,str2,..')

# Data Property

## AUTO_INCREMENT
Auto +1 and it must be defined as a key(primary key)

## BINARY(Case Sensitive)
比较BINARY 列时将区分大小小方排序，默认不区分

## ZEROFILL

	int(5) ZEROFILL

## NATIONAL
该列使用默认字符集，兼容考虑
It can only be used to CHAR/STRING.

