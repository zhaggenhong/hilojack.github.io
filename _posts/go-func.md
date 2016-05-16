---
layout: page
title:
category: blog
description:
---
# Preface

# parameters
When two or more *consecutive(连续) named* function parameters share a type, you can omit the type from all but the last.

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

## Named return values
A return statement `without` arguments returns the named return values.
This is known as a `"naked"` return.

	package main

	import "fmt"

	func split(sum int) (x, y int) {
		x = sum * 4 / 9
		y = sum - x
		return
	}

	func main() {
		fmt.Println(split(17))
	}

## Short variable declarations
Inside a function, the `:=` short assignment statement can be used in place of `a var declaration` with `implicit type`.

Outside a function, every statement begins with a `keyword (var, func, and so on)` and so the `:=` construct is not available.

```
	package main

	import "fmt"

	func main() {
		var i, j int = 1, 2
		k := 3
		c, python, java := true, false, "no!"

		fmt.Println(i, j, k, c, python, java)
	}
```
