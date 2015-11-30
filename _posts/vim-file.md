---
layout: page
title:	
category: blog
description: 
---
# Preface

- filetype
- Directory
- file
- file open and exit

# Filetype
Overview:					*:filetype-overview*

	command				detection	plugin		indent ~
	:filetype on			on		unchanged	unchanged
	:filetype off			off		unchanged	unchanged
	:filetype plugin on		on		on		unchanged
	:filetype plugin off		unchanged	off		unchanged
	:filetype indent on		on		unchanged	on
	:filetype indent off		unchanged	unchanged	off
	:filetype plugin indent on	on		on		on
	:filetype plugin indent off	unchanged	off		off

To see the current status, type: >

	:filetype

If the ":syntax on" command is used, the file type detection is installed too(implied).  There is no need
to do ":filetype on" after ":syntax on".

## set get filetype
:setf :setfiletype

	au! BufNewFile,BufRead *.bat,*.sys setf dosbatch

get filetype:

	:set filetype?
	filetype=mkd
	filetype=javascript
	filetype=python

# Directory

## Path
> :h expand()
> :h filename-modifiers

	:echom expand('%')					"relative path
	:echom expand('%:p')				"absolute path
	:echom expand('~/.vimrc')				"absolute path
	:echom fnamemodify('foo.txt', ':p') "Absolute path

:h filename-modifiers:

	:. reduce file name to current directory
	:r root of the file name(without extension)
	:e extension of file name

### wildcard
> :h wildcard

	?	matches one character
	*	matches anything, including nothing
	**	matches anything, including nothing, recurses into directories
	[abc]	match 'a', 'b' or 'c'

Example: >

	:n **/*.txt

When non-wildcard characters are used these are only matched in the first
directory.  Example: >

	:n /usr/inc**/*.h

### filename

	fnameescape('./a b')
	simplify("./dir/.././/file/") == "./file/"

## list file

	:echo globpath('.', '*')
	:echo globpath('.', '*.txt')
	:echo globpath('.', '**')		"recursively list
	:echo globpath('.', '**/*.txt')	"recursively list

return list

	:echo split(globpath('.', '*'), "\n")

# File

## file attribute

	filereadable(expand('~/.vimrc'))
	exec 'source '.fnameescape('~/.vimrc')

## line num

	line('$')

## getline

	getline({lnum}) "get line of lnum
	getline('.')

## indent

	function! IndentLevel(lnum)
		return indent(a:lnum) / &shiftwidth
	endfunction

## append

	setlocal buftype=nofile
    call append(lnum, split('a,b', ','))

:read

	:[lnum]r <file>
	:[lnum]r !cmd

# Open, save, exit

	noremap <C-q> :qa<CR>

## Open

	:e <file>
	:f[ile] <new_file_name>

> refer `# Window`

### Read Only 只读

	vim -R a.txt　#加!也可以修改
	vim -M a.txt #不可编辑
	//也可以临时更改为可读
	:set write
	:set modifiable

### Encrypt 加密

	#打开加密文件
	:vim -x a.txt
	:X #设置加密key
	:set key= #取消加密key
	#加密编辑时，不产生交换文件
	vim -x -n a.txt or :set noswapfile

### Binary File 二进制文件

	vim -b a.mp3
	#以十六进制显示
	:%!xxd or :set display=uhex

## Write
append(without change reload)

	:[range]w >> 
	:[range]w >> {file}

## Exit

	ZZ 保存退出
	ZQ 不保存退出

保存

	:wa 全部保存
	:wqa 

write when changes exist

	:x :xit :exit 

不保存

	:q! 不保存退出
	:qa! 全部不保存退出

## Init

	"make vim save and load the folding of the document each time it loads"
	"also places the cursor in the last place that it was left."
	au BufWinLeave * mkview
	au BufWinEnter * silent loadview

### modeline 模式行
你可能只希望打开某一部分文件时，执行一些初始化．这样放到vimrc或者ftplugin中都不合适．这时，可以采用模式行——把初始化命令放到文件本身

比如你希望打开a.txt时，tab是7个，那么在a.txt前n行或者后n行中写入模式行（n=modelines）:

	/* vim:set tabstop=7: */

模式行的格式是(两边可放任意字符，建议是/* */)：
	
	any-text vim:set name=value ...: any-text

模式行的数量可以限制，比如：

	:set modelines=10

### 打开时光标位置
这个功能需要viminfo的支持(用vim --version | grep viminfo查看)

	'. "normal mode 返回上次离开时编辑的位置(见按保存状态移动)
	'" "normal mode 返回上次离开时光标所在的位置

	autocmd BufReadPost *
		 \ if line("'\"") <= line("$") |
		 \   exe "normal! g`\"" |
		 \ endif

## 缓冲与备份

### backup
make a backup before overwriting file

	:set backup
	:set backupext=.bak ＃备份文件的扩展名

	:w >>a.txt " 向文件中追加内容
	:sav a.txt :saveas a.txt(当前文件名也更改了)
	:.,$w file.txt #保存当前行到最后一行

### noswapfile

	:set noswapfile
	vim -x -n a.txt " 不缓存
