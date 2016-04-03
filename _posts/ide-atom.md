---
layout: page
title:
category: blog
description:
---
# Preface

# amp(Atom Package Management)

## amp config

	apm config set http_proxy  "http://127.0.0.1:8080"
	apm config set https_proxy  "http://127.0.0.1:8080"
	cat ~/.atom/.apmrc

## amp version

	$ apm --version
	apm  1.6.0
	npm  2.13.3
	node 0.10.40
	python 2.7.11
	git 2.5.4

## amp install

	apm install markdown-writer

## apm 404
在安装包时，可能会遇到G*F*W 问题：

	$ apm install markdown-writer
	Installing markdown-writer to /Users/hilojack/.atom/packages
	gyp info it worked if it ends with ok
	gyp info using node-gyp@2.0.2
	gyp info using node@0.10.40 | darwin | x64
	gyp http GET https://atom.io/download/atom-shell/v0.34.5/node-v0.34.5.tar.gz
	gyp http 404 https://atom.io/download/atom-shell/v0.34.5/node-v0.34.5.tar.gz
	gyp WARN install got an error, rolling back install
	gyp ERR! install error
	gyp ERR! stack Error: 404 response downloading https://atom.io/download/atom-shell/v0.34.5/node-v0.34.5.tar.gz

参考: https://github.com/atom/apm/issues/174
