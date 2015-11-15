---
layout: page
title:	
category: blog
description: 
---
# Preface

# autocmd
Syntax:

	:au[tocmd] [group] {event} {pat} [nested] {cmd}
			Add {cmd} to the list of commands that Vim will
			execute automatically on {event} for a file matching
			{pat} |autocmd-patterns|.

	:autocmd BufNewFile * :write
         ^          ^ ^
         |          | |
         |          | The command to run.
         |          |
         |          A "pattern" to filter the event.
         |
         The "event" to watch for.

## event
> :help autocmd-events

### io event

	BufNewFile
	BufRead
	BufWrite
	BufWritePre "same as BufWrite
	BufWritePre,BufRead 
	
Example:

	:autocmd BufWritePre *.html :normal gg=G
	:autocmd BufWritePre *.html normal gg=G

### FileType event
excute when filetype is set or vim starts

	:autocmd FileType javascript nnoremap <buffer> <localleader>c I//<esc>
	:autocmd FileType python nnoremap <buffer> <localleader>c I//<esc>

Refer to: [vim-filetype](/p/vim-filetype)

## pattern

	*
	*.txt
	javascript

multi pattern:

	:autocmd FileType c,cpp  map <buffer> <leader><space> :w<cr>:make<cr>

## group
> :help autocmd-groups

To solove the problem: duplicated autocmd group, we can use `autocmd!` to clear current group

	:augroup testgroup
	:    autocmd!
	:    autocmd BufWrite * :echom "Cats"
	:    autocmd BufWrite * :echom "Dogs"
	:augroup END

Vim automatically wraps the contents of ftdetect/*.vim files in autocommand groups for you, so you don't need to use an autocommand group like we usually would.

## specical characters
*specical characters* in the `:autocmd` arguments are *not expanded*. The only excetion is `sfile`:

	:au BufNewFile,BufRead *.html so <sfile>:h/html.vim

# command

## command exist

	echo has('autocmd')

## execute command

	:execute "echom 'Hello, world!'"
	
multi args:

	:exec arg1   arg2 ...
		:arg1 arg2 arg3 will be concat to command seperated by "<space>"
	:exe "echom" "'Hello," "world!'"

## multi command

	:cmd1|cmd2|cmd3

## silent
To disalbe command's output.

	:silent echo "abc"
	:silent !echo "abc"
	:silent execute "normal! .."

## normal command

	:normal gg=G

	"If then [!] is given, mapping will not be used
	:normal! gg=G

normal does not recognize special charactors, but `execute` with *double quotes* can!

	"wrong
	:normal! gg/a<cr>
	:execute 'normal! gg/a<cr>' 
	:execute "normal! gg/a<cr>" 
	:execute 'normal! gg/a\r'

	"right
	:execute "normal! gg/a\<cr>"
	:execute "normal! gg/a\r" 

Refer to `:h expr-quote` and [vim-var](/p/vim-var)


# External and vim command

## command shell
Syntax:

	vim [options] filelist
	-	stdin
	+[num]	put cursor to line num or last line

	+/{pat}

	+{command} or 
	-c {command}
			   vim -c 'cmd1|cmd2|cmd3'
	--cmd {command}
			   Like using "-c", but the command is executed just before  process-
			   ing any vimrc file. 

	-S {file}   {file}  will  be sourced after the first file has been read.  This
			   is equivalent to -c "source {file}".   {file}  cannot  start  with
			   '-'.   If {file} is omitted "Session.vim" is used (only works when
			   -S is the last argument).

    -s <scriptin>    The characters in file <scriptin> is interpreted as if you typed them , like `:source! {scriptin}`
	-w <scriptout>
		All the characters you  typed are recorded in <scriptout>(append), this file can be used with `:source`
	-W <scriptpout>
		Like -w, but existing file is overwritten.

	-u {path_to_vimrc}
		All the other initializations are skipped.
		vim -u NONE " No init file.

## external bang command
Syntax:

	# 替换式
	!{motion}{program} "program处理完了后，替换motion
	:[range]!{program} "program处理完了后，替换range

	# 非替换式(仅:w)
	:[range]{cmd} !{program} "自带cmd 处理内容时，重定向文件描述符到stdin，program 读取stdin
	:!{program} 直接读取stdin

eg:

	#比如我想让一到５行的内容经过sort,这个命令会从normal进入到命令行
	!5Gsort<enter>
	:.,5!sort<enter>

	#!!{program} #此时motion为!代表当前行
	!!wc
	:.!wc

### pipe to external bang cmd

	:[range]w[rite] [++opt] !{cmd}
	:w !sudo tee %

## system

	echo system('ls')
	echo system('wc -c', stdin)
	echo system('cat', 'sth.')

# user command

## define command

	:com [{attr}...] {cmd} {rep}
			Define a user command.  The name of the command is
			{cmd} and its replacement text is {rep}.  The command's
			attributes (see below) are {attr}.  
### args
Argument:

	-nargs=0    No arguments are allowed (the default)
	-nargs=1    Exactly one argument is required, it includes spaces 
	-nargs=*    Any number of arguments are allowed (0, 1, or many),
		    separated by white space
	-nargs=?    0 or 1 arguments are allowed
	-nargs=+    Arguments must be supplied, but any number are allowed

Specifically, "s:var" will use the script-local variable in the script where the command was
defined, not where it is invoked!  

single args`<args>`:

    " script1.vim: >
	command -nargs=1 Error echoerr <args>

	" script2.vim: >
	source script1.vim
	Error "E215"
        E215

Multi args`<f-args>`:

    " Mycmd arg1 arg2
    " function Myfunc(myarg1, myarg2)
    com -nargs=* Mycmd call Myfunc(<f-args>)

quote args `<q-args>`(raw)

    com -nargs=1 E1 echo <q-args>
    E1 "E215"a b
        "E215"a b

## list
list all user commmand

	:com
	:com {cmd}
	:verbose command {cmd}

## delete

	"clear all
	:comc :comclear

	"delete command
	:delc {cmd}

# Reference
- [learnvim]

[learnvim]: http://learnvimscriptthehardway.stevelosh.com/chapters/12.html
