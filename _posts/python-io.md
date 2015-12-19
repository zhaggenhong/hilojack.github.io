---
layout: page
title:
category: blog
description:
---
# Preface

# IO

## url

	from urllib import urlopen

readline:

	urlp = urlopen(url);
	while(line = urlp.readline()){

	}

readlines:

	for line in urlopen(url).readlines():
		print line.strip()

## argv

	from sys import argv # import module "sys" and objects: argv
	script, arg1, arg2 = argv

	import sys;
	sys.argv

## input

	str = input()
	str = input("Input some string:")

## Directory

	from os.path import exists
	exists(file)

## File

	print open('a.php').read(); //cat a.php

### Open File

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

### read and write
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

### file position
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

#### line position

	function seek_line(f, line){
		f.seek(0);
		while(line-- >0){
			f.getline();
		}
	}

