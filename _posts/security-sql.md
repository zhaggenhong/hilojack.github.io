---
layout: page
title:	sql injection
category: blog
description: 
---
# Preface

# Blind SQL
不显示错误，盲注

# Wide Charset Injection
在GBK宽字节中

	%df'
	%df%27
	%df%5c%27 # %df%5c 是gbk 中是一个汉字

%5C 是\

##  multi-byte characters
(multi-byte). Here are three examples of character tables/encoding:

 - ISO-8859-1/LATIN1 (French)
 - UTF-8 (international, multi-byte)
 - SJIS, BIG5, GBK, GB18030, UHC (Asian, multi-byte)

A SQL string can contain the tick character. For example:

	SELECT ... WHERE var = 'abc\'def';
	SELECT ... WHERE var = 'abc''def';

The mysql_real_escape_string() function adds backslash or doubles ticks.

The first vulnerability affects the `mysql_real_escape_string()` function family which does not reject invalid multi-byte characters:

For example, in UTF-8, the `"0xC8 ' ' attackersql"` or `"0xC8 \ ' attackersql"` string is converted to "one_character ' attackersql" (ignore spaces). So, the query:

	  SELECT ... WHERE var = ' mysql_real_escape_string("0xC8 ' attackersql") '
	become :
	  SELECT ... WHERE var = ' 0xC8 ' ' attackersql '
	  SELECT ... WHERE var = 'one_character ' attackersql'

An attacker can therefore inject the attackersql command.
