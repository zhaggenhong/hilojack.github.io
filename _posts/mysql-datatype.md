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

## 不要用null
Null 列的缺点:

- 难以优化索引：Mysql难以优化引用可空列查询，它会使索引、索引统计和值更加复杂。
- 更多的空间：可空列需要更多的存储空间，还需要mysql内部进行特殊处理。
> NULL columns require additional space in the rowto record whether their values are NULL. For MyISAM tables, each NULL columntakes one bit extra, rounded up to the nearest byte.”

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

