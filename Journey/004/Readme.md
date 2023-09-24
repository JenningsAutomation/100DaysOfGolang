
![placeholder image](basics3.png)

# Still Learning the Basics

## Introduction

✍️ In this installment, I'm continuing to go through the fundamentals. I'm working my way down all the code on the Go website. Currently, I'm in Go by Example.

## Prerequisite

✍️ Have Go installed and an IDE

## Try yourself

### Step 1 — If/Else
 
- You can have an if statement without an else.

- A statement can precede conditionals; any variables declared in this statement are available in the current and all subsequent branches.

- Note that you don’t need parentheses around conditions in Go, but that the braces are required.

- There is no ternary if in Go, so you’ll need to use a full if statement even for basic conditions.

```
package main

import "fmt"

func main() {

    if 7%2 == 0 {
        fmt.Println("7 is even")
    } else {
        fmt.Println("7 is odd")
    }

    if 8%4 == 0 {
        fmt.Println("8 is divisible by 4")
    }

    if num := 9; num < 0 {
        fmt.Println(num, "is negative")
    } else if num < 10 {
        fmt.Println(num, "has 1 digit")
    } else {
        fmt.Println(num, "has multiple digits")
    }
}
```
output: 
```	
$ go run if-else.go 
7 is odd
8 is divisible by 4
9 has 1 digit
```


### Step 2 — Switch

Switch statements express conditionals across many branches.

Here’s a basic switch.
```
    i := 2
    fmt.Print("Write ", i, " as ")
    switch i {
    case 1:
        fmt.Println("one")
    case 2:
        fmt.Println("two")
    case 3:
        fmt.Println("three")
    }
```

You can use commas to separate multiple expressions in the same case statement. We use the optional default case in this example as well.

```
switch time.Now().Weekday() {
    case time.Saturday, time.Sunday:
        fmt.Println("It's the weekend")
    default:
        fmt.Println("It's a weekday")
    }
```
switch without an expression is an alternate way to express if/else logic. Here we also show how the case expressions can be non-constants.

```
    t := time.Now()
    switch {
    case t.Hour() < 12:
        fmt.Println("It's before noon")
    default:
        fmt.Println("It's after noon")
    }
```

A type switch compares types instead of values. You can use this to discover the type of an interface value. In this example, the variable t will have the type corresponding to its clause.

```
    whatAmI := func(i interface{}) {
        switch t := i.(type) {
        case bool:
            fmt.Println("I'm a bool")
        case int:
            fmt.Println("I'm an int")
        default:
            fmt.Printf("Don't know type %T\n", t)
        }
    }
    whatAmI(true)
    whatAmI(1)
    whatAmI("hey")
```
Complete Example:
```
package main

import (
    "fmt"
    "time"
)

func main() {

    i := 2
    fmt.Print("Write ", i, " as ")
    switch i {
    case 1:
        fmt.Println("one")
    case 2:
        fmt.Println("two")
    case 3:
        fmt.Println("three")
    }

    switch time.Now().Weekday() {
    case time.Saturday, time.Sunday:
        fmt.Println("It's the weekend")
    default:
        fmt.Println("It's a weekday")
    }

    t := time.Now()
    switch {
    case t.Hour() < 12:
        fmt.Println("It's before noon")
    default:
        fmt.Println("It's after noon")
    }

    whatAmI := func(i interface{}) {
        switch t := i.(type) {
        case bool:
            fmt.Println("I'm a bool")
        case int:
            fmt.Println("I'm an int")
        default:
            fmt.Printf("Don't know type %T\n", t)
        }
    }
    whatAmI(true)
    whatAmI(1)
    whatAmI("hey")
}
```

output:
```	
$ go run switch.go 
Write 2 as two
It's a weekday
It's after noon
I'm a bool
I'm an int
Don't know type string
```

### Step 3 — Arrays

In Go, an array is a numbered sequence of elements of a specific length. In typical Go code, slices are much more common; arrays are useful in some special scenarios.

Here we create an array a that will hold exactly 5 ints. The type of elements and length are both part of the array’s type. By default an array is zero-valued, which for ints means 0s.

```
    var a [5]int
    fmt.Println("emp:", a)
```
We can set a value at an index using the array[index] = value syntax, and get a value with array[index]
```
    a[4] = 100
    fmt.Println("set:", a)
    fmt.Println("get:", a[4])
```
The builtin len returns the length of an array.
```
fmt.Println("len:", len(a))
```
Use this syntax to declare and initialize an array in one line.
```	
    b := [5]int{1, 2, 3, 4, 5}
    fmt.Println("dcl:", b)
```
Array types are one-dimensional, but you can compose types to build multi-dimensional data structures.
```
    var twoD [2][3]int
    for i := 0; i < 2; i++ {
        for j := 0; j < 3; j++ {
            twoD[i][j] = i + j
        }
    }
    fmt.Println("2d: ", twoD)
```

Note that arrays appear in the form [v1 v2 v3 ...] when printed with fmt.Println.
output:
```
$ go run arrays.go
emp: [0 0 0 0 0]
set: [0 0 0 0 100]
get: 100
len: 5
dcl: [1 2 3 4 5]
2d:  [[0 1 2] [1 2 3]]

```

### Step 5 — Slices

Slices are an important data type in Go, giving a more powerful interface to sequences than arrays.

Unlike arrays, slices are typed only by the elements they contain (not the number of elements). An uninitialized slice equals to nil and has length 0.

```
    var s []string
    fmt.Println("uninit:", s, s == nil, len(s) == 0)
```
To create an empty slice with non-zero length, use the builtin make. Here we make a slice of strings of length 3 (initially zero-valued). By default a new slice’s capacity is equal to its length; if we know the slice is going to grow ahead of time, it’s possible to pass a capacity explicitly as an additional parameter to make.
```
    s = make([]string, 3)
    fmt.Println("emp:", s, "len:", len(s), "cap:", cap(s))
```
We can set and get just like with arrays.
```
    s[0] = "a"
    s[1] = "b"
    s[2] = "c"
    fmt.Println("set:", s)
    fmt.Println("get:", s[2])
```
len returns the length of the slice as expected.
```
fmt.Println("len:", len(s))
```
In addition to these basic operations, slices support several more that make them richer than arrays. One is the builtin append, which returns a slice containing one or more new values. Note that we need to accept a return value from append as we may get a new slice value.

```
    s = append(s, "d")
    s = append(s, "e", "f")
    fmt.Println("apd:", s)
```

Slices can also be copy’d. Here we create an empty slice c of the same length as s and copy into c from s.
```
    c := make([]string, len(s))
    copy(c, s)
    fmt.Println("cpy:", c)
```
Slices support a “slice” operator with the syntax slice[low:high]. For example, this gets a slice of the elements s[2], s[3], and s[4].

```
    l := s[2:5]
    fmt.Println("sl1:", l)
```

This slices up to (but excluding) s[5].
```
    l = s[:5]
    fmt.Println("sl2:", l)
```

And this slices up from (and including) s[2].
```
    l = s[2:]
    fmt.Println("sl3:", l)
```

We can declare and initialize a variable for slice in a single line as well.
```   
    t := []string{"g", "h", "i"}
    fmt.Println("dcl:", t) 
```
The slices package contains a number of useful utility functions for slices.
```
    t2 := []string{"g", "h", "i"}
    if slices.Equal(t, t2) {
        fmt.Println("t == t2")
    }
```
Slices can be composed into multi-dimensional data structures. The length of the inner slices can vary, unlike with multi-dimensional arrays.
```
    twoD := make([][]int, 3)
    for i := 0; i < 3; i++ {
        innerLen := i + 1
        twoD[i] = make([]int, innerLen)
        for j := 0; j < innerLen; j++ {
            twoD[i][j] = i + j
        }
    }
    fmt.Println("2d: ", twoD)
```

Note that while slices are different types than arrays, they are rendered similarly by fmt.Println.
```
$ go run slices.go
uninit: [] true true
emp: [  ] len: 3 cap: 3
set: [a b c]
get: c
len: 3
apd: [a b c d e f]
cpy: [a b c d e f]
sl1: [c d e]
sl2: [a b c d e]
sl3: [c d e f]
dcl: [g h i]
t == t2
2d:  [[0] [1 2] [2 3 4]]
```

Complete program:
```
package main

import (
    "fmt"
    "slices"
)

func main() {

    var s []string
    fmt.Println("uninit:", s, s == nil, len(s) == 0)

    s = make([]string, 3)
    fmt.Println("emp:", s, "len:", len(s), "cap:", cap(s))

    s[0] = "a"
    s[1] = "b"
    s[2] = "c"
    fmt.Println("set:", s)
    fmt.Println("get:", s[2])

    fmt.Println("len:", len(s))

    s = append(s, "d")
    s = append(s, "e", "f")
    fmt.Println("apd:", s)

    c := make([]string, len(s))
    copy(c, s)
    fmt.Println("cpy:", c)

    l := s[2:5]
    fmt.Println("sl1:", l)

    l = s[:5]
    fmt.Println("sl2:", l)

    l = s[2:]
    fmt.Println("sl3:", l)

    t := []string{"g", "h", "i"}
    fmt.Println("dcl:", t)

    t2 := []string{"g", "h", "i"}
    if slices.Equal(t, t2) {
        fmt.Println("t == t2")
    }

    twoD := make([][]int, 3)
    for i := 0; i < 3; i++ {
        innerLen := i + 1
        twoD[i] = make([]int, innerLen)
        for j := 0; j < innerLen; j++ {
            twoD[i][j] = i + j
        }
    }
    fmt.Println("2d: ", twoD)
}
```

output:
```
$ go run slices.go
uninit: [] true true
emp: [  ] len: 3 cap: 3
set: [a b c]
get: c
len: 3
apd: [a b c d e f]
cpy: [a b c d e f]
sl1: [c d e]
sl2: [a b c d e]
sl3: [c d e f]
dcl: [g h i]
t == t2
2d:  [[0] [1 2] [2 3 4]]
```

### Step 6 — Maps
Maps are Go’s built-in associative data type (sometimes called hashes or dicts in other languages).

To create an empty map, use the builtin make: make(map[key-type]val-type).
```
m := make(map[string]int)
```
Set key/value pairs using typical name[key] = val syntax.
```
    m["k1"] = 7
    m["k2"] = 13
```
Printing a map with e.g. fmt.Println will show all of its key/value pairs.
```
fmt.Println("map:", m)
```
Get a value for a key with name[key].
```
    v1 := m["k1"]
    fmt.Println("v1:", v1)
```
If the key doesn’t exist, the zero value of the value type is returned.
```
    v3 := m["k3"]
    fmt.Println("v3:", v3)
```
The builtin len returns the number of key/value pairs when called on a map.
```
fmt.Println("len:", len(m))
```
The builtin delete removes key/value pairs from a map.
```
    delete(m, "k2")
    fmt.Println("map:", m)
```
To remove all key/value pairs from a map, use the clear builtin.
```
    clear(m)
    fmt.Println("map:", m)
```
The optional second return value when getting a value from a map indicates if the key was present in the map. This can be used to disambiguate between missing keys and keys with zero values like 0 or "". Here we didn’t need the value itself, so we ignored it with the blank identifier _.
```
    _, prs := m["k2"]
    fmt.Println("prs:", prs)
```

You can also declare and initialize a new map in the same line with this syntax.
```
    n := map[string]int{"foo": 1, "bar": 2}
    fmt.Println("map:", n)
```
The maps package contains a number of useful utility functions for maps.
```
	
    n2 := map[string]int{"foo": 1, "bar": 2}
    if maps.Equal(n, n2) {
        fmt.Println("n == n2")
    }
```
complete code:
```
package main

import (
    "fmt"
    "maps"
)

func main() {

    m := make(map[string]int)

    m["k1"] = 7
    m["k2"] = 13

    fmt.Println("map:", m)

    v1 := m["k1"]
    fmt.Println("v1:", v1)

    v3 := m["k3"]
    fmt.Println("v3:", v3)

    fmt.Println("len:", len(m))

    delete(m, "k2")
    fmt.Println("map:", m)

    clear(m)
    fmt.Println("map:", m)

    _, prs := m["k2"]
    fmt.Println("prs:", prs)

    n := map[string]int{"foo": 1, "bar": 2}
    fmt.Println("map:", n)

    n2 := map[string]int{"foo": 1, "bar": 2}
    if maps.Equal(n, n2) {
        fmt.Println("n == n2")
    }
}
```
Note that maps appear in the form map[k:v k:v] when printed with fmt.Println.

output:
```
$ go run maps.go 
map: map[k1:7 k2:13]
v1: 7
v3: 0
len: 2
map: map[k1:7]
map: map[]
prs: false
map: map[bar:2 foo:1]
n == n2
```

### Step 7 — Range

range iterates over elements in a variety of data structures. Let’s see how to use range with some of the data structures we’ve already learned.
Here we use range to sum the numbers in a slice. Arrays work like this too.
```
    nums := []int{2, 3, 4}
    sum := 0
    for _, num := range nums {
        sum += num
    }
    fmt.Println("sum:", sum)
```
range on arrays and slices provides both the index and value for each entry. Above we didn’t need the index, so we ignored it with the blank identifier _. Sometimes we actually want the indexes though.
```
    for i, num := range nums {
        if num == 3 {
            fmt.Println("index:", i)
        }
    }
```
range on map iterates over key/value pairs.
```	
    kvs := map[string]string{"a": "apple", "b": "banana"}
    for k, v := range kvs {
        fmt.Printf("%s -> %s\n", k, v)
    }
```
range can also iterate over just the keys of a map.
```
    for k := range kvs {
        fmt.Println("key:", k)
    }
```	
range on strings iterates over Unicode code points. The first value is the starting byte index of the rune and the second the rune itself. See Strings and Runes for more details.
```
    for i, c := range "go" {
        fmt.Println(i, c)
    }
```
```
package main

import "fmt"

func main() {

    nums := []int{2, 3, 4}
    sum := 0
    for _, num := range nums {
        sum += num
    }
    fmt.Println("sum:", sum)

    for i, num := range nums {
        if num == 3 {
            fmt.Println("index:", i)
        }
    }

    kvs := map[string]string{"a": "apple", "b": "banana"}
    for k, v := range kvs {
        fmt.Printf("%s -> %s\n", k, v)
    }

    for k := range kvs {
        fmt.Println("key:", k)
    }

    for i, c := range "go" {
        fmt.Println(i, c)
    }
}
```


output:
```
$ go run range.go
sum: 9
index: 1
a -> apple
b -> banana
key: a
key: b
0 103
1 111
```

### Step 8 — Functions
Functions are central in Go. We’ll learn about functions with a few different examples.
Here’s a function that takes two ints and returns their sum as an int.

Go requires explicit returns, i.e. it won’t automatically return the value of the last expression.
```
func plus(a int, b int) int {
        return a + b
}

```
When you have multiple consecutive parameters of the same type, you may omit the type name for the like-typed parameters up to the final parameter that declares the type.
```
func plusPlus(a, b, c int) int {
    return a + b + c
}
```

Call a function just as you’d expect, with name(args).
```
    res := plus(1, 2)
    fmt.Println("1+2 =", res)
    res = plusPlus(1, 2, 3)
    fmt.Println("1+2+3 =", res)
```

complete code:
```
package main

import "fmt"

func plus(a int, b int) int {

    return a + b
}

func plusPlus(a, b, c int) int {
    return a + b + c
}

func main() {

    res := plus(1, 2)
    fmt.Println("1+2 =", res)

    res = plusPlus(1, 2, 3)
    fmt.Println("1+2+3 =", res)
}
```

output:
```
$ go run functions.go 
1+2 = 3
1+2+3 = 6

```

## ☁️ Outcome

✍️ You can't escape the fundamentals. Coming from doing mostly python, I'm equating some of the python commands to Go. 

## Next Steps

✍️ I'll keep working my way down Go by example

## Social Proof

✍️ [Toot](https://mastodon.social/@code_sentinel/111120701493253828)
