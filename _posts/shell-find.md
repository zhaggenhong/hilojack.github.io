---
layout: page
title:	
category: blog
description: 
---
# Preface

	find -maxdepth 1 -name '*' -inum 1324 -exec rm {} \;
	find -maxdepth 1 -name '*' -inum 1324 -exec mv {} {}.txt \;
	//Note! There is a space before terminating ";" or "+"
	find . -name .DS_Store -exec rm {} +
	//du -d 1

# -iname
case insensitive

	-iname '*abc*'

# -perm

	754(rwxr-xr--) a.txt
	find . -perm 755 # 755 == 754 false
	find . -perm +4000 # 4000 & 0754 true
	find . -perm -4000 # (4000 & 0754) == 4000 false
	find . \(! -perm -0754 -o -perm 766 \)

# Logic Operators
The primaries may be combined using the following operators.  The operators are listed in order of decreasing precedence.

     ( expression )
             This evaluates to true if the parenthesized expression evaluates to true.

     ! expression
     -not expression
             This is the unary NOT operator.  It evaluates to true if the expression is false.

     -false  Always false.
     -true   Always true.

     expression -and expression
     expression -a expression
     expression expression
             The -and operator is the logical AND operator.  As it is implied by the juxtaposition of two expressions it does not have to be specified.  The expression evaluates to
             true if both expressions are true.  The second expression is not evaluated if the first expression is false.

     expression -or expression
     expression -o expression
             The -or operator is the logical OR operator.  The expression evaluates to true if either the first or the second expression is true.  The second expression is not evaluated if the first expression is true.

For example:

	#! -path exclude dir
	find . -type f -name '*.php' ! -path "./dir1/*" ! -path "./scripts/*"
	find . -type f ! -path './[.|nbproject|todo]*'

    ! expr1 expr2
    //equal to 
    ( ! expr1 ) expr2 
    //not equal to
    ! ( expr1 expr2 )

# match path

## -regex
支持BRE 正则，和ERE 正则(-E 参数)

	find . -regex ".*/[abc]/.*" 
	find -E . -regex ".*/(word1|word2)/.+" 

## -path
支持wildcard

	-path pattern
	-path './[dir1|dir2]/*'

## exclude path
`-prune` stops find from descending into a directory. 

	# -prune does not exclude the directory itself but its contents. So you should use `-print` to print
	find . -path ./dir -prune -o -name '*.txt' -print
	find . -name dirname -prune -o -name '*.txt' -print

    # prune multiple directories
	find . \( -path ./dir1 -o -path ./dir2 \) -prune -o -name '*.txt' -print
    # prune with not
	find . ! \( \(-path ./dir1 -o path ./dir2 \) -prune \) -name \*.txt

Just specifying `-not -path` cannot stop find from descending into the skipped directory, so `-not -path` will work but it is not the best method:

    "not work
	find . ! -path './dir1/*' -name '*'

# match file
Match dirname and file via `-name`

	# wildcard
	find dir1 dir2 -name "*.py"
