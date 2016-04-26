---
layout: page
title:
category: blog
description:
---
# Preface

# User

	import os, stat

    uid = os.getuid()
    euid = os.geteuid()
    gid = os.getgid()
    egid = os.getegid()

# Directory

## glob files
Starting with Python version 3.5, the glob module supports the `"**"` directive for recursive
(which is parsed only if you pass `recursive flag`):

	import glob
	glob.glob(pathname, *, recursive=False); # list
	glob.iglob(pathname, *, recursive=False); # iterator

	for filename in glob.iglob('src/**/*.c', recursive=True):
		print(filename)

If you need an list, just use `glob.glob` instead of `glob.iglob`.

## access permision
Use the real uid/gid to test for access to a path.

	access(path, mode, *, dir_fd=None, effective_ids=False, follow_symlinks=True)
		mode
			Operating-system mode bitfield.  Can be F_OK to test existence,
			or the inclusive-OR of R_OK, W_OK, and X_OK.

example:

	if os.access(file, os.R_OK):
		# Do something
	else:
		if not os.access(file, os.R_OK):
			print(file, "is not readable")
		else:
			print("Something went wrong with file/dir", file)
		break

### chmod

	os.chmod(path, mode)
	os.chown(path, uid, gid)

## file.stat

	import os
	st = os.stat(path)
    if st.st_uid == euid:
        return st.st_mode & stat.S_IRUSR != 0

## isfile

	os.path.isfile("bob.txt") # Does bob.txt exist?  Is it a file, or a directory?
	os.path.isdir("bob")

## path

	import os
	path = os.path.dirname(amodule.__file__)
	path = os.path.abspath(amodule.__file__)

## mkdir/rename/delete/

	os.mkdir(dir,mode=511)
		os.makedirs(newDir/dir);//recursive

### move file:

	os.rename(fileA, existedDir/fileB)
	os.move(fileA, existedDir/fileB)
		os.renames('dirA', 'newDir/DirB')//auto create dir
		os.renames('a', 'c/')// rename a to c (rmdir c if c is existed)

> `shutil.move()` uses `os.rename()` if the destination is on the current filesystem. 
Otherwise, `shutil.move()` copies the source to destination using `shutil.copy2()` and then removes the source.

### copy file
`copy2` is also often useful, it preserves the `original modification and access info (mtime and atime)` in the file metadata.

	shutil.copyfile(src, existedDir/dst)
	shutil.copy2(src, existedDir/dst)

### remove file

	os.rmdir(empty_dir);	//empty dir
		shutil.rmtree(dir_only)
	os.remove(file_only);	//file_only

## current directory

	os.getcwd()
	os.chdir(path)
	os.chroot(path)

## mkfile

	os.mkfifo(path, mode=438, *, dir_fd=None)
        Create a "fifo" (a POSIX named pipe).

# File

	print open('a.php').read(); //cat a.php

stdin file


	import fileinput
	for line in fileinput.input():
		print(line)

## IOError

	import glob, os
	try:
		for file in glob.glob("\\*.txt"):
			with open(file) as fp:
				# do something with file
	except IOError:
		print("could not read", file)


## Open File

	fp = open(filename[, mode])
	mode:
		'r' 'w' 'a'
		append a '+' to the mode to allow simultaneous reading and writing
			'r+' read and write
			'w+' empty old content and write
			'a+' read and append
		append a 'U' to the mode open a file for input with universal new line support('\r', '\n', '\r\n')
			 'U' cannot be combined with 'w' or '+' mode.
	fp.close()

### binary

	>>> f = open('/Users/michael/test.jpg', 'rb')
	>>> f.read()
	b'\xff\xd8\xff\xe1\x00\x18Exif\x00\x00...' # 十六进制表示的字节

### gbk
gbk 且忽略非法编码

	>>> f = open('/Users/michael/gbk.txt', 'r', encoding='gbk', errors='ignore')

## read and write
> pydoc file

	fp.read()	Return file content.
	fp.readline()	Return just one line of text file.
	fp.write(string)	Writes string to file(start at file position)
	fp.writeline(string)	Writes string to file(start at file position)
	fp.truncate(...)
		truncate([size]) -> None.  Truncate the file to at most size bytes.
		Size defaults to the current file position, as returned by tell().

Example:

	fp = open('a.txt')
	print fp.read()
	fp.close();

## close file
由于文件读写时都有可能产生IOError，一旦出错，后面的f.close()就不会调用。
所以，为了保证无论是否出错都能正确地关闭文件，我们可以使用try ... finally来实现：

	try:
		f = open('/path/to/file', 'r')
		print(f.read())
	finally:
		if f:
			f.close()

但是每次都这么写实在太繁琐，所以，Python引入了with语句来自动帮我们调用close()方法：

	with open('/path/to/file', 'r') as f:
		print(f.read())

## file position
`pydoc file`

	fp.tell(...)
		tell() -> current file position, an integer (may be a long integer).
	seek(...)
		seek(offset[, whence]) -> None.  Move to new file position.
		Optional argument whence defaults to:
			0 (offset from start of file, offset should be >= 0);
			1 (move relative to current position, positive or negative),
			2 (move relative to end of file, usually negative, although many platforms allow seeking beyond the end of a file).
			If the file is opened in text mode, only offsets returned by tell() are legal.  Use of other offsets causes undefined behavior.

### line position

	function seek_line(f, line){
		f.seek(0);
		while(line-- >0){
			f.getline();
		}
	}

# File-like Object
像open()函数返回的这种有个read()方法的对象，在Python中统称为file-like Object。除了file外，还可以是内存的字节流，网络流，自定义流等等。file-like Object不要求从特定类继承，只要写个read()方法就行。

StringIO就是在内存中创建的file-like Object，常用作临时缓冲。

# StringIO和BytesIO

## StringIO
StringIO顾名思义就是在内存中读写str。


要把str写入StringIO，我们需要先创建一个StringIO，然后，像文件一样写入即可：

	>>> from io import StringIO
	>>> f = StringIO()
	>>> f.write('hello')
	5
	>>> f.write(' ')
	1
	>>> f.write('world!')
	6
	>>> print(f.getvalue())
	hello world!

### init

	>>> from io import StringIO
	>>> f = StringIO('Hello!\nHi!\nGoodbye!')

### read

	f.read()
	f.readline()
	f.readlines()

## BytesIO
StringIO操作的只能是str，如果要操作二进制数据，就需要使用BytesIO。

	>>> from io import BytesIO
	>>> f = BytesIO()
	>>> f.write('中文'.encode('utf-8'))
	6

	>>> print(f.getvalue())
	b'\xe4\xb8\xad\xe6\x96\x87'

string 与 bytes 相互转换

	>>> '123€20'.encode('utf-8')
	b'123\xe2\x82\xac20'
	>>> b'\xe2\x82\xac20'.decode('utf-8')
	'€20'
	>>> '123'.encode('utf-8')

或者：

	>>> bytes([0, 1, 97])
	b'\x00\x01a'

# url

	from urllib import urlopen

readline:

	urlp = urlopen(url);
	while(line = urlp.readline()){

	}

readlines:

	for line in urlopen(url).readlines():
		print line.strip()

# pipe

    for l in sys.stdin.readlines():
        sys.stdout.write(l[::-1])

# argv

	from sys import argv # import module "sys" and objects: argv
	script, arg1, arg2 = argv

	import sys;
	sys.argv

# input

	str = input()
	str = input("Input some string:")

