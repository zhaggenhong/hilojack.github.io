---
layout: page
title:
category: blog
description:
---
# Preface

# parameters
When two or more *consecutive(连续) named* function parameters share a type, you can omit the type from all but the last.

In this example, we shortened

	x int, y int
to

	x, y int

Example

	package main
	import "fmt"

	func add(x, y int) int {
		return x + y
	}

	func main() {
		fmt.Println(add(42, 13))
	}

# Multiple results
A function can return any number of results.

The `swap` function returns two strings.

	package main
	import "fmt"

	func swap(x, y string) (string, string) {
		return y, x
	}

	func main() {
		a, b := swap("hello", "world")
		fmt.Println(a, b)
	}
