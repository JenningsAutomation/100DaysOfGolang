
![back to school](gopher_schoolboy.png)

# More Basics

## Introduction

✍️ I continue with Go by example. This day I worked on code for Multiple return values, Variadic Functions, Closures, Recursion and Pointers

## Prerequisite

✍️ Have Go installed and the ability to read.


## Cloud Research

- ✍️ This is all from the Golang website.

### Step 1 — Multiple Return Values

Go has built-in support for multiple return values. This feature is used often in idiomatic Go, for example to return both result and error values from a function.
The (int, int) in this function signature shows that the function returns 2 ints.
Here we use the 2 different return values from the call with multiple assignment.
If you only want a subset of the returned values, use the blank identifier _.
Accepting a variable number of arguments is another nice feature of Go functions; we’ll look at this next.

```
package main

import "fmt"

func vals() (int, int) {
    return 3, 7
}

func main() {

    a, b := vals()
    fmt.Println(a)
    fmt.Println(b)

    _, c := vals()
    fmt.Println(c)
}
```
result:
```
$ go run multiple-return-values.go
3
7
7
```

### Step 2 — Variadic Functions

Variadic functions can be called with any number of trailing arguments. For example, fmt.Println is a common variadic function.
Here’s a function that will take an arbitrary number of ints as arguments.
```
package main
import "fmt"
func sum(nums ...int) {
    fmt.Print(nums, " ")
    total := 0
```
Within the function, the type of nums is equivalent to []int. We can call len(nums), iterate over it with range, etc.
```
for _, num := range nums {
        total += num
    }
    fmt.Println(total)
}
```


Variadic functions can be called in the usual way with individual arguments.

```
func main() {
    sum(1, 2)
    sum(1, 2, 3)
```
If you already have multiple args in a slice, apply them to a variadic function using func(slice...) like this.

```
    nums := []int{1, 2, 3, 4}
    sum(nums...)
}
```


complete code:
```
package main

import "fmt"

func sum(nums ...int) {
    fmt.Print(nums, " ")
    total := 0

    for _, num := range nums {
        total += num
    }
    fmt.Println(total)
}

func main() {

    sum(1, 2)
    sum(1, 2, 3)

    nums := []int{1, 2, 3, 4}
    sum(nums...)
}
```
result:
```
$ go run variadic-functions.go 
[1 2] 3
[1 2 3] 6
[1 2 3 4] 10
```

### Step 3 — Closures

Go supports anonymous functions, which can form closures. Anonymous functions are useful when you want to define a function inline without having to name it. This is like a lambda function in python

```
// Go supports [_anonymous functions_](https://en.wikipedia.org/wiki/Anonymous_function),
// which can form <a href="https://en.wikipedia.org/wiki/Closure_(computer_science)"><em>closures</em></a>.
// Anonymous functions are useful when you want to define
// a function inline without having to name it.

package main

import "fmt"

// This function `intSeq` returns another function, which
// we define anonymously in the body of `intSeq`. The
// returned function _closes over_ the variable `i` to
// form a closure.
func intSeq() func() int {
	i := 0
	return func() int {
		i++
		return i
	}
}

func main() {

	// We call `intSeq`, assigning the result (a function)
	// to `nextInt`. This function value captures its
	// own `i` value, which will be updated each time
	// we call `nextInt`.
	nextInt := intSeq()

	// See the effect of the closure by calling `nextInt`
	// a few times.
	fmt.Println(nextInt())
	fmt.Println(nextInt())
	fmt.Println(nextInt())

	// To confirm that the state is unique to that
	// particular function, create and test a new one.
	newInts := intSeq()
	fmt.Println(newInts())
}

```

Output:

```
	
$ go run closures.go
1
2
3
1
```
### Step 4 — Closures
Go supports recursive functions. Here’s a classic example.
```
// Go supports
// <a href="https://en.wikipedia.org/wiki/Recursion_(computer_science)"><em>recursive functions</em></a>.
// Here's a classic example.

package main

import "fmt"

// This `fact` function calls itself until it reaches the
// base case of `fact(0)`.
func fact(n int) int {
	if n == 0 {
		return 1
	}
	return n * fact(n-1)
}

func main() {
	fmt.Println(fact(7))

	// Closures can also be recursive, but this requires the
	// closure to be declared with a typed `var` explicitly
	// before it's defined.
	var fib func(n int) int

	fib = func(n int) int {
		if n < 2 {
			return n
		}

		// Since `fib` was previously declared in `main`, Go
		// knows which function to call with `fib` here.
		return fib(n-1) + fib(n-2)
	}

	fmt.Println(fib(7))
}
```
output:
```
5040
13

Program exited.
```
### Step 5 - Pointers
Go supports pointers, allowing you to pass references to values and records within your program.
```
// Go supports <em><a href="https://en.wikipedia.org/wiki/Pointer_(computer_programming)">pointers</a></em>,
// allowing you to pass references to values and records
// within your program.

package main

import "fmt"

// We'll show how pointers work in contrast to values with
// 2 functions: `zeroval` and `zeroptr`. `zeroval` has an
// `int` parameter, so arguments will be passed to it by
// value. `zeroval` will get a copy of `ival` distinct
// from the one in the calling function.
func zeroval(ival int) {
	ival = 0
}

// `zeroptr` in contrast has an `*int` parameter, meaning
// that it takes an `int` pointer. The `*iptr` code in the
// function body then _dereferences_ the pointer from its
// memory address to the current value at that address.
// Assigning a value to a dereferenced pointer changes the
// value at the referenced address.
func zeroptr(iptr *int) {
	*iptr = 0
}

func main() {
	i := 1
	fmt.Println("initial:", i)

	zeroval(i)
	fmt.Println("zeroval:", i)

	// The `&i` syntax gives the memory address of `i`,
	// i.e. a pointer to `i`.
	zeroptr(&i)
	fmt.Println("zeroptr:", i)

	// Pointers can be printed too.
	fmt.Println("pointer:", &i)
}
```


output:
```
initial: 1
zeroval: 1
zeroptr: 0
pointer: 0xc000012028

Program exited.
```

## ☁️  Outcome

✍️ I continue to learn the basics

## Next Steps

✍️ More Go by example, starting with Strings and runes

## Social Proof

[Toot](https://mastodon.social/@code_sentinel/111168947794018945)
