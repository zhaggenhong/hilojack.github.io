---
layout: page
title:
category: blog
description:
---
# Preface

Ctrl-tab

	switch tab

Note: SecureCRT's default copy and paste hotkeys,

	COMMAND+SHIFT+C (copy)
	COMMAND+SHIFT+V (paste)

https://www.vandyke.com/support/tips/nl102004.html

Session->Terminal->Anti-idle:

	send-string 1800
	send-protocol 180

# check
Bash:

	shopt | grep checkwinsize

If you don't get

	checkwinsize    on

then activate it with

	shopt -s checkwinsize
	# deactive
	shopt -u checkwinsize
