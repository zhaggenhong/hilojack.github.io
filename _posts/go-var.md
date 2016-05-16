---
layout: page
title:
category: blog
description:
---
# Preface

# Variables
The var statement declares a list of variables

	package main

	import "fmt"

	var c, python, java bool

	func main() {
		var i int
		fmt.Println(i, c, python, java)
	}

## initializers
A var declaration can include initializers, one per variable.

> If an initializer is present, the type can be omitted; the variable will take the type of the initializer.

	var i, j int = 1, 2

### Zero values
Variables declared without an explicit initial value are given their zero value.

The zero value is:

	0 for numeric types,
	false for the boolean type, and
	"" (the empty string) for strings.

```
var i int
var f float64
var b bool
```

## Short variable declarations
Inside a function, the `:=` short assignment statement can be used in place of `a var declaration with implicit type`.

Outside a function, every statement begins with a keyword (var, func, and so on) and so the `:= construct` is not available.

	package main

	import "fmt"

	func main() {
		var i, j int = 1, 2
		k := 3
		c, python, java := true, false, "no!"

		fmt.Println(i, j, k, c, python, java)
	}

# Datatype
Go's basic types are

```
bool

string

int  int8  int16  int32  int64
uint uint8 uint16 uint32 uint64 uintptr

byte // alias for uint8

rune // alias for int32
     // represents a Unicode code point

float32 float64

complex64 complex128
```

The `int, uint, and uintptr` types are usually `32 bits wide` on 32-bit systems and `64 bits wide` on 64-bit systems.
When you need an integer value you should use `int` unless you have a specific reason to use a sized or unsigned integer type.

```
package main

import (
	"fmt"
	"math/cmplx"
)

var (
	ToBe   bool       = false
	MaxInt uint64     = 1<<64 - 1
	z      complex128 = cmplx.Sqrt(-5 + 12i)
)

func main() {
	const f = "%T(%v)\n"
	fmt.Printf(f, ToBe, ToBe)
	fmt.Printf(f, MaxInt, MaxInt)
	fmt.Printf(f, z, z)
}
```
## Type conversions
The expression T(v) converts the value v to the type T.

Some numeric conversions:

```
var i int = 42
var f float64 = float64(i)
var u uint = uint(f)
```

Or, put more simply:

```
i := 42
f := float64(i)
u := uint(f)
```

Unlike in C, in Go assignment between items of different type requires an explicit conversion. Try removing the float64 or uint conversions in the example and see what happens.

	var x, y int = 3, 4
	var f float64 = math.Sqrt(float64(x*x + y*y))
	var z uint = uint(f)
	fmt.Println(x, y, z)

When the right hand side of the declaration is typed, the new variable is of that same type:

```
	var i int
	j := i // j is an int
```

But when the right hand side contains an untyped numeric constant, the new variable may be an int, float64, or complex128 depending on the precision of the constant:

	i := 42           // int
	f := 3.142        // float64
	g := 0.867 + 0.5i // complex128

Example

```
func main() {
	v := 42 // change me!
	fmt.Printf("v is of type %T\n", v)
}
```

# Const

	package main

	import "fmt"

	const Pi = 3.14

	func main() {
		const World = "世界"
		fmt.Println("Hello", World)
		fmt.Println("Happy", Pi, "Day")

		const Truth = true
		fmt.Println("Go rules?", Truth)
	}
