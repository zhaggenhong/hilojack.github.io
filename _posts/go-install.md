---
layout: page
title:
category: blog
description:
---
# Preface

# install

	brew install go

# hello
1. set workspace

	export GOPATH=$HOME/www/go
	# 默认的
	GOROOT=/usr/local/Cellar/go/1.6.2/libexec/

cat www/go/src/A/hello/hello.go

	package main

	import "fmt"

	func main() {
		fmt.Printf("hello, world\n")
	}

compile:
会默认去$GOROOT/$GOPATH 找`src/A/hello/hello.go`

	$ go install A/hello


得到目录树

	.
	├── pkg			编译生成的包文件
	├── bin 		生成的可执行文件
	│   └── hello
	└── src
		└── A
			└── hello
				└── hello.go
	
