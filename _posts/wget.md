---
layout: page
title:	wget
category: blog
description:
---
# header

	-U mozilla
		set User-Agent
	-e
		robots=off if you don’t want wget to obey by the robots.txt file

## cookie

	-b, --cookie
		-b "NAME1=VALUE1; NAME2=VALUE2"
		-b 'a.cookie'
		 If  no  '='  symbol  is  used  in the line, it is treated as a filename to use to read previously stored cookie lines from.

	-c, --cookie-jar <file name>
	  Specify to which file you want curl to write all cookies after a completed operation.
	  Curl writes all cookies `previously read` from a `specified file`  as  well  as all cookies `received from remote server`.
	-c -
		write to oupput

	curl -D- -b a.txt -c a.txt url.com
	curl -D- -b a.txt -c - url.com #for debug

# output

## output file

	-O file
	-O -
		specify output file as stdin
	-o logfile
		log all messages to logfile
	-r
		Recursive download.(全站, 不需要-p)
	-p
		This option causes Wget to download all the files that are necessary to properly display a given HTML page.
		This includes such things as inlined images, sounds, and ref-erenced stylesheets.

Example: Dump Site

	wget --random-wait -r -p -e robots=off -U mozilla http://pcottle.github.io/learnGitBranching/
	-q , --quiet
	-v, --verbose
		Makes the  fetching more verbose


## output log

	-o log_file
		logs the output

## dump header

	-D file
		dump header to file
	-D-
		dump to output

# limit

### wait time
	--random-wait to let wget chose a random number of seconds to wait, avoid get into black list.

### limit rate
	--limit-rate=20k limits the rate at which it downloads files.
