**Learning the Fundamentals**
![gopherschool](schooltime3.png)

# More Go by Example

## Introduction

✍️ Today was a good day. I covered Stateful Goroutines,Sorting,Sorting by Functions,Panic,Defer,Recover,String Functions,String Formatting,Text Templates,Regular Expressions, and JSON

## Prerequisite

✍️ HAve golang installed

## Research
- ✍️ Just following Go by example on the Golang website

### Step 1 — Stateful Goroutines
In the previous example we used explicit locking with mutexes to synchronize access to shared state across multiple goroutines. Another option is to use the built-in synchronization features of goroutines and channels to achieve the same result. This channel-based approach aligns with Go’s ideas of sharing memory by communicating and having each piece of data owned by exactly 1 goroutine.

```
package main

import (
	"fmt"
	"math/rand"
	"sync/atomic"
	"time"
)

// In this example our state will be owned by a single
// goroutine. This will guarantee that the data is never
// corrupted with concurrent access. In order to read or
// write that state, other goroutines will send messages
// to the owning goroutine and receive corresponding
// replies. These `readOp` and `writeOp` `struct`s
// encapsulate those requests and a way for the owning
// goroutine to respond.
type readOp struct {
	key  int
	resp chan int
}
type writeOp struct {
	key  int
	val  int
	resp chan bool
}

func main() {

	// As before we'll count how many operations we perform.
	var readOps uint64
	var writeOps uint64

	// The `reads` and `writes` channels will be used by
	// other goroutines to issue read and write requests,
	// respectively.
	reads := make(chan readOp)
	writes := make(chan writeOp)

	// Here is the goroutine that owns the `state`, which
	// is a map as in the previous example but now private
	// to the stateful goroutine. This goroutine repeatedly
	// selects on the `reads` and `writes` channels,
	// responding to requests as they arrive. A response
	// is executed by first performing the requested
	// operation and then sending a value on the response
	// channel `resp` to indicate success (and the desired
	// value in the case of `reads`).
	go func() {
		var state = make(map[int]int)
		for {
			select {
			case read := <-reads:
				read.resp <- state[read.key]
			case write := <-writes:
				state[write.key] = write.val
				write.resp <- true
			}
		}
	}()

	// This starts 100 goroutines to issue reads to the
	// state-owning goroutine via the `reads` channel.
	// Each read requires constructing a `readOp`, sending
	// it over the `reads` channel, and then receiving the
	// result over the provided `resp` channel.
	for r := 0; r < 100; r++ {
		go func() {
			for {
				read := readOp{
					key:  rand.Intn(5),
					resp: make(chan int)}
				reads <- read
				<-read.resp
				atomic.AddUint64(&readOps, 1)
				time.Sleep(time.Millisecond)
			}
		}()
	}

	// We start 10 writes as well, using a similar
	// approach.
	for w := 0; w < 10; w++ {
		go func() {
			for {
				write := writeOp{
					key:  rand.Intn(5),
					val:  rand.Intn(100),
					resp: make(chan bool)}
				writes <- write
				<-write.resp
				atomic.AddUint64(&writeOps, 1)
				time.Sleep(time.Millisecond)
			}
		}()
	}

	// Let the goroutines work for a second.
	time.Sleep(time.Second)

	// Finally, capture and report the op counts.
	readOpsFinal := atomic.LoadUint64(&readOps)
	fmt.Println("readOps:", readOpsFinal)
	writeOpsFinal := atomic.LoadUint64(&writeOps)
	fmt.Println("writeOps:", writeOpsFinal)
}

```
output:
```
readOps: 100100
writeOps: 10010

Program exited.
```
### Step 2 — Sorting
Go’s slices package implements sorting for builtins and user-defined types. We’ll look at sorting for builtins first.
```
package main

import (
	"fmt"
	"slices"
)

func main() {

	// Sorting functions are generic, and work for any
	// _ordered_ built-in type. For a list of ordered
	// types, see [cmp.Ordered](https://pkg.go.dev/cmp#Ordered).
	strs := []string{"c", "a", "b"}
	slices.Sort(strs)
	fmt.Println("Strings:", strs)

	// An example of sorting `int`s.
	ints := []int{7, 2, 4}
	slices.Sort(ints)
	fmt.Println("Ints:   ", ints)

	// We can also use the `slices` package to check if
	// a slice is already in sorted order.
	s := slices.IsSorted(ints)
	fmt.Println("Sorted: ", s)
}

```
output:
```
Strings: [a b c]
Ints:    [2 4 7]
Sorted:  true

Program exited.
```
### Step 3 — Sorting by Functions

```
// Sometimes we'll want to sort a collection by something
// other than its natural order. For example, suppose we
// wanted to sort strings by their length instead of
// alphabetically. Here's an example of custom sorts
// in Go.

package main

import (
	"cmp"
	"fmt"
	"slices"
)

func main() {
	fruits := []string{"peach", "banana", "kiwi"}

	// We implement a comparison function for string
	// lengths. `cmp.Compare` is helpful for this.
	lenCmp := func(a, b string) int {
		return cmp.Compare(len(a), len(b))
	}

	// Now we can call `slices.SortFunc` with this custom
	// comparison function to sort `fruits` by name length.
	slices.SortFunc(fruits, lenCmp)
	fmt.Println(fruits)

	// We can use the same technique to sort a slice of
	// values that aren't built-in types.
	type Person struct {
		name string
		age  int
	}

	people := []Person{
		Person{name: "Jax", age: 37},
		Person{name: "TJ", age: 25},
		Person{name: "Alex", age: 72},
	}

	// Sort `people` by age using `slices.SortFunc`.
	//
	// Note: if the `Person` struct is large,
	// you may want the slice to contain `*Person` instead
	// and adjust the sorting function accordingly. If in
	// doubt, [benchmark](testing-and-benchmarking)!
	slices.SortFunc(people,
		func(a, b Person) int {
			return cmp.Compare(a.age, b.age)
		})
	fmt.Println(people)
}

```
output:
```
[kiwi peach banana]
[{TJ 25} {Jax 37} {Alex 72}]

Program exited.
```
### Step 4 — Panic

```
// A `panic` typically means something went unexpectedly
// wrong. Mostly we use it to fail fast on errors that
// shouldn't occur during normal operation, or that we
// aren't prepared to handle gracefully.

package main

import "os"

func main() {

	// We'll use panic throughout this site to check for
	// unexpected errors. This is the only program on the
	// site designed to panic.
	panic("a problem")

	// A common use of panic is to abort if a function
	// returns an error value that we don't know how to
	// (or want to) handle. Here's an example of
	// `panic`king if we get an unexpected error when creating a new file.
	_, err := os.Create("/tmp/file")
	if err != nil {
		panic(err)
	}
}

```
output:
```
./prog.go:21:2: unreachable code

Go vet failed.

panic: a problem

goroutine 1 [running]:
main.main()
	/tmp/sandbox4284204573/prog.go:15 +0x25

Program exited.
```
### Step 5 — Defer
```
// _Defer_ is used to ensure that a function call is
// performed later in a program's execution, usually for
// purposes of cleanup. `defer` is often used where e.g.
// `ensure` and `finally` would be used in other languages.

package main

import (
	"fmt"
	"os"
)

// Suppose we wanted to create a file, write to it,
// and then close when we're done. Here's how we could
// do that with `defer`.
func main() {

	// Immediately after getting a file object with
	// `createFile`, we defer the closing of that file
	// with `closeFile`. This will be executed at the end
	// of the enclosing function (`main`), after
	// `writeFile` has finished.
	f := createFile("/tmp/defer.txt")
	defer closeFile(f)
	writeFile(f)
}

func createFile(p string) *os.File {
	fmt.Println("creating")
	f, err := os.Create(p)
	if err != nil {
		panic(err)
	}
	return f
}

func writeFile(f *os.File) {
	fmt.Println("writing")
	fmt.Fprintln(f, "data")

}

func closeFile(f *os.File) {
	fmt.Println("closing")
	err := f.Close()
	// It's important to check for errors when closing a
	// file, even in a deferred function.
	if err != nil {
		fmt.Fprintf(os.Stderr, "error: %v\n", err)
		os.Exit(1)
	}
}

```
output:
```
creating
writing
closing

Program exited.
```
### Step 6 — Recover
```
// Go makes it possible to _recover_ from a panic, by
// using the `recover` built-in function. A `recover` can
// stop a `panic` from aborting the program and let it
// continue with execution instead.

// An example of where this can be useful: a server
// wouldn't want to crash if one of the client connections
// exhibits a critical error. Instead, the server would
// want to close that connection and continue serving
// other clients. In fact, this is what Go's `net/http`
// does by default for HTTP servers.

package main

import "fmt"

// This function panics.
func mayPanic() {
	panic("a problem")
}

func main() {
	// `recover` must be called within a deferred function.
	// When the enclosing function panics, the defer will
	// activate and a `recover` call within it will catch
	// the panic.
	defer func() {
		if r := recover(); r != nil {
			// The return value of `recover` is the error raised in
			// the call to `panic`.
			fmt.Println("Recovered. Error:\n", r)
		}
	}()

	mayPanic()

	// This code will not run, because `mayPanic` panics.
	// The execution of `main` stops at the point of the
	// panic and resumes in the deferred closure.
	fmt.Println("After mayPanic()")
}

```
output:
```
Recovered. Error:
 a problem

Program exited.
```
### Step 7 — String Functions
```
// The standard library's `strings` package provides many
// useful string-related functions. Here are some examples
// to give you a sense of the package.

package main

import (
	"fmt"
	s "strings"
)

// We alias `fmt.Println` to a shorter name as we'll use
// it a lot below.
var p = fmt.Println

func main() {

	// Here's a sample of the functions available in
	// `strings`. Since these are functions from the
	// package, not methods on the string object itself,
	// we need pass the string in question as the first
	// argument to the function. You can find more
	// functions in the [`strings`](https://pkg.go.dev/strings)
	// package docs.
	p("Contains:  ", s.Contains("test", "es"))
	p("Count:     ", s.Count("test", "t"))
	p("HasPrefix: ", s.HasPrefix("test", "te"))
	p("HasSuffix: ", s.HasSuffix("test", "st"))
	p("Index:     ", s.Index("test", "e"))
	p("Join:      ", s.Join([]string{"a", "b"}, "-"))
	p("Repeat:    ", s.Repeat("a", 5))
	p("Replace:   ", s.Replace("foo", "o", "0", -1))
	p("Replace:   ", s.Replace("foo", "o", "0", 1))
	p("Split:     ", s.Split("a-b-c-d-e", "-"))
	p("ToLower:   ", s.ToLower("TEST"))
	p("ToUpper:   ", s.ToUpper("test"))
}

```
output:
```
Contains:   true
Count:      2
HasPrefix:  true
HasSuffix:  true
Index:      1
Join:       a-b
Repeat:     aaaaa
Replace:    f00
Replace:    f0o
Split:      [a b c d e]
ToLower:    test
ToUpper:    TEST

Program exited.
```

### Step 8 — String Formatting
```
// Go offers excellent support for string formatting in
// the `printf` tradition. Here are some examples of
// common string formatting tasks.

package main

import (
	"fmt"
	"os"
)

type point struct {
	x, y int
}

func main() {

	// Go offers several printing "verbs" designed to
	// format general Go values. For example, this prints
	// an instance of our `point` struct.
	p := point{1, 2}
	fmt.Printf("struct1: %v\n", p)

	// If the value is a struct, the `%+v` variant will
	// include the struct's field names.
	fmt.Printf("struct2: %+v\n", p)

	// The `%#v` variant prints a Go syntax representation
	// of the value, i.e. the source code snippet that
	// would produce that value.
	fmt.Printf("struct3: %#v\n", p)

	// To print the type of a value, use `%T`.
	fmt.Printf("type: %T\n", p)

	// Formatting booleans is straight-forward.
	fmt.Printf("bool: %t\n", true)

	// There are many options for formatting integers.
	// Use `%d` for standard, base-10 formatting.
	fmt.Printf("int: %d\n", 123)

	// This prints a binary representation.
	fmt.Printf("bin: %b\n", 14)

	// This prints the character corresponding to the
	// given integer.
	fmt.Printf("char: %c\n", 33)

	// `%x` provides hex encoding.
	fmt.Printf("hex: %x\n", 456)

	// There are also several formatting options for
	// floats. For basic decimal formatting use `%f`.
	fmt.Printf("float1: %f\n", 78.9)

	// `%e` and `%E` format the float in (slightly
	// different versions of) scientific notation.
	fmt.Printf("float2: %e\n", 123400000.0)
	fmt.Printf("float3: %E\n", 123400000.0)

	// For basic string printing use `%s`.
	fmt.Printf("str1: %s\n", "\"string\"")

	// To double-quote strings as in Go source, use `%q`.
	fmt.Printf("str2: %q\n", "\"string\"")

	// As with integers seen earlier, `%x` renders
	// the string in base-16, with two output characters
	// per byte of input.
	fmt.Printf("str3: %x\n", "hex this")

	// To print a representation of a pointer, use `%p`.
	fmt.Printf("pointer: %p\n", &p)

	// When formatting numbers you will often want to
	// control the width and precision of the resulting
	// figure. To specify the width of an integer, use a
	// number after the `%` in the verb. By default the
	// result will be right-justified and padded with
	// spaces.
	fmt.Printf("width1: |%6d|%6d|\n", 12, 345)

	// You can also specify the width of printed floats,
	// though usually you'll also want to restrict the
	// decimal precision at the same time with the
	// width.precision syntax.
	fmt.Printf("width2: |%6.2f|%6.2f|\n", 1.2, 3.45)

	// To left-justify, use the `-` flag.
	fmt.Printf("width3: |%-6.2f|%-6.2f|\n", 1.2, 3.45)

	// You may also want to control width when formatting
	// strings, especially to ensure that they align in
	// table-like output. For basic right-justified width.
	fmt.Printf("width4: |%6s|%6s|\n", "foo", "b")

	// To left-justify use the `-` flag as with numbers.
	fmt.Printf("width5: |%-6s|%-6s|\n", "foo", "b")

	// So far we've seen `Printf`, which prints the
	// formatted string to `os.Stdout`. `Sprintf` formats
	// and returns a string without printing it anywhere.
	s := fmt.Sprintf("sprintf: a %s", "string")
	fmt.Println(s)

	// You can format+print to `io.Writers` other than
	// `os.Stdout` using `Fprintf`.
	fmt.Fprintf(os.Stderr, "io: an %s\n", "error")
}

```
ouput:
```
struct1: {1 2}
struct2: {x:1 y:2}
struct3: main.point{x:1, y:2}
type: main.point
bool: true
int: 123
bin: 1110
char: !
hex: 1c8
float1: 78.900000
float2: 1.234000e+08
float3: 1.234000E+08
str1: "string"
str2: "\"string\""
str3: 6865782074686973
pointer: 0xc0000a2000
width1: |    12|   345|
width2: |  1.20|  3.45|
width3: |1.20  |3.45  |
width4: |   foo|     b|
width5: |foo   |b     |
sprintf: a string
io: an error

Program exited.
```
### Step 9 — Text Templating
```
// Go offers built-in support for creating dynamic content or showing customized
// output to the user with the `text/template` package. A sibling package
// named `html/template` provides the same API but has additional security
// features and should be used for generating HTML.

package main

import (
	"os"
	"text/template"
)

func main() {

	// We can create a new template and parse its body from
	// a string.
	// Templates are a mix of static text and "actions" enclosed in
	// `{{...}}` that are used to dynamically insert content.
	t1 := template.New("t1")
	t1, err := t1.Parse("Value is {{.}}\n")
	if err != nil {
		panic(err)
	}

	// Alternatively, we can use the `template.Must` function to
	// panic in case `Parse` returns an error. This is especially
	// useful for templates initialized in the global scope.
	t1 = template.Must(t1.Parse("Value: {{.}}\n"))

	// By "executing" the template we generate its text with
	// specific values for its actions. The `{{.}}` action is
	// replaced by the value passed as a parameter to `Execute`.
	t1.Execute(os.Stdout, "some text")
	t1.Execute(os.Stdout, 5)
	t1.Execute(os.Stdout, []string{
		"Go",
		"Rust",
		"C++",
		"C#",
	})

	// Helper function we'll use below.
	Create := func(name, t string) *template.Template {
		return template.Must(template.New(name).Parse(t))
	}

	// If the data is a struct we can use the `{{.FieldName}}` action to access
	// its fields. The fields should be exported to be accessible when a
	// template is executing.
	t2 := Create("t2", "Name: {{.Name}}\n")

	t2.Execute(os.Stdout, struct {
		Name string
	}{"Jane Doe"})

	// The same applies to maps; with maps there is no restriction on the
	// case of key names.
	t2.Execute(os.Stdout, map[string]string{
		"Name": "Mickey Mouse",
	})

	// if/else provide conditional execution for templates. A value is considered
	// false if it's the default value of a type, such as 0, an empty string,
	// nil pointer, etc.
	// This sample demonstrates another
	// feature of templates: using `-` in actions to trim whitespace.
	t3 := Create("t3",
		"{{if . -}} yes {{else -}} no {{end}}\n")
	t3.Execute(os.Stdout, "not empty")
	t3.Execute(os.Stdout, "")

	// range blocks let us loop through slices, arrays, maps or channels. Inside
	// the range block `{{.}}` is set to the current item of the iteration.
	t4 := Create("t4",
		"Range: {{range .}}{{.}} {{end}}\n")
	t4.Execute(os.Stdout,
		[]string{
			"Go",
			"Rust",
			"C++",
			"C#",
		})
}

```
outcome:
```
Value: some text
Value: 5
Value: [Go Rust C++ C#]
Name: Jane Doe
Name: Mickey Mouse
yes 
no 
Range: Go Rust C++ C# 

Program exited.
```
### Step 10 — Regular Expressions
```
// Go offers built-in support for [regular expressions](https://en.wikipedia.org/wiki/Regular_expression).
// Here are some examples of  common regexp-related tasks
// in Go.

package main

import (
	"bytes"
	"fmt"
	"regexp"
)

func main() {

	// This tests whether a pattern matches a string.
	match, _ := regexp.MatchString("p([a-z]+)ch", "peach")
	fmt.Println(match)

	// Above we used a string pattern directly, but for
	// other regexp tasks you'll need to `Compile` an
	// optimized `Regexp` struct.
	r, _ := regexp.Compile("p([a-z]+)ch")

	// Many methods are available on these structs. Here's
	// a match test like we saw earlier.
	fmt.Println(r.MatchString("peach"))

	// This finds the match for the regexp.
	fmt.Println(r.FindString("peach punch"))

	// This also finds the first match but returns the
	// start and end indexes for the match instead of the
	// matching text.
	fmt.Println("idx:", r.FindStringIndex("peach punch"))

	// The `Submatch` variants include information about
	// both the whole-pattern matches and the submatches
	// within those matches. For example this will return
	// information for both `p([a-z]+)ch` and `([a-z]+)`.
	fmt.Println(r.FindStringSubmatch("peach punch"))

	// Similarly this will return information about the
	// indexes of matches and submatches.
	fmt.Println(r.FindStringSubmatchIndex("peach punch"))

	// The `All` variants of these functions apply to all
	// matches in the input, not just the first. For
	// example to find all matches for a regexp.
	fmt.Println(r.FindAllString("peach punch pinch", -1))

	// These `All` variants are available for the other
	// functions we saw above as well.
	fmt.Println("all:", r.FindAllStringSubmatchIndex(
		"peach punch pinch", -1))

	// Providing a non-negative integer as the second
	// argument to these functions will limit the number
	// of matches.
	fmt.Println(r.FindAllString("peach punch pinch", 2))

	// Our examples above had string arguments and used
	// names like `MatchString`. We can also provide
	// `[]byte` arguments and drop `String` from the
	// function name.
	fmt.Println(r.Match([]byte("peach")))

	// When creating global variables with regular
	// expressions you can use the `MustCompile` variation
	// of `Compile`. `MustCompile` panics instead of
	// returning an error, which makes it safer to use for
	// global variables.
	r = regexp.MustCompile("p([a-z]+)ch")
	fmt.Println("regexp:", r)

	// The `regexp` package can also be used to replace
	// subsets of strings with other values.
	fmt.Println(r.ReplaceAllString("a peach", "<fruit>"))

	// The `Func` variant allows you to transform matched
	// text with a given function.
	in := []byte("a peach")
	out := r.ReplaceAllFunc(in, bytes.ToUpper)
	fmt.Println(string(out))
}

```
outcome:
```
true
true
peach
idx: [0 5]
[peach ea]
[0 5 1 3]
[peach punch pinch]
all: [[0 5 1 3] [6 11 7 9] [12 17 13 15]]
[peach punch]
true
regexp: p([a-z]+)ch
a <fruit>
a PEACH

Program exited.
```
### Step 11 — JSON
```
// Go offers built-in support for JSON encoding and
// decoding, including to and from built-in and custom
// data types.

package main

import (
	"encoding/json"
	"fmt"
	"os"
)

// We'll use these two structs to demonstrate encoding and
// decoding of custom types below.
type response1 struct {
	Page   int
	Fruits []string
}

// Only exported fields will be encoded/decoded in JSON.
// Fields must start with capital letters to be exported.
type response2 struct {
	Page   int      `json:"page"`
	Fruits []string `json:"fruits"`
}

func main() {

	// First we'll look at encoding basic data types to
	// JSON strings. Here are some examples for atomic
	// values.
	bolB, _ := json.Marshal(true)
	fmt.Println(string(bolB))

	intB, _ := json.Marshal(1)
	fmt.Println(string(intB))

	fltB, _ := json.Marshal(2.34)
	fmt.Println(string(fltB))

	strB, _ := json.Marshal("gopher")
	fmt.Println(string(strB))

	// And here are some for slices and maps, which encode
	// to JSON arrays and objects as you'd expect.
	slcD := []string{"apple", "peach", "pear"}
	slcB, _ := json.Marshal(slcD)
	fmt.Println(string(slcB))

	mapD := map[string]int{"apple": 5, "lettuce": 7}
	mapB, _ := json.Marshal(mapD)
	fmt.Println(string(mapB))

	// The JSON package can automatically encode your
	// custom data types. It will only include exported
	// fields in the encoded output and will by default
	// use those names as the JSON keys.
	res1D := &response1{
		Page:   1,
		Fruits: []string{"apple", "peach", "pear"}}
	res1B, _ := json.Marshal(res1D)
	fmt.Println(string(res1B))

	// You can use tags on struct field declarations
	// to customize the encoded JSON key names. Check the
	// definition of `response2` above to see an example
	// of such tags.
	res2D := &response2{
		Page:   1,
		Fruits: []string{"apple", "peach", "pear"}}
	res2B, _ := json.Marshal(res2D)
	fmt.Println(string(res2B))

	// Now let's look at decoding JSON data into Go
	// values. Here's an example for a generic data
	// structure.
	byt := []byte(`{"num":6.13,"strs":["a","b"]}`)

	// We need to provide a variable where the JSON
	// package can put the decoded data. This
	// `map[string]interface{}` will hold a map of strings
	// to arbitrary data types.
	var dat map[string]interface{}

	// Here's the actual decoding, and a check for
	// associated errors.
	if err := json.Unmarshal(byt, &dat); err != nil {
		panic(err)
	}
	fmt.Println(dat)

	// In order to use the values in the decoded map,
	// we'll need to convert them to their appropriate type.
	// For example here we convert the value in `num` to
	// the expected `float64` type.
	num := dat["num"].(float64)
	fmt.Println(num)

	// Accessing nested data requires a series of
	// conversions.
	strs := dat["strs"].([]interface{})
	str1 := strs[0].(string)
	fmt.Println(str1)

	// We can also decode JSON into custom data types.
	// This has the advantages of adding additional
	// type-safety to our programs and eliminating the
	// need for type assertions when accessing the decoded
	// data.
	str := `{"page": 1, "fruits": ["apple", "peach"]}`
	res := response2{}
	json.Unmarshal([]byte(str), &res)
	fmt.Println(res)
	fmt.Println(res.Fruits[0])

	// In the examples above we always used bytes and
	// strings as intermediates between the data and
	// JSON representation on standard out. We can also
	// stream JSON encodings directly to `os.Writer`s like
	// `os.Stdout` or even HTTP response bodies.
	enc := json.NewEncoder(os.Stdout)
	d := map[string]int{"apple": 5, "lettuce": 7}
	enc.Encode(d)
}

```
outcome:
```
true
1
2.34
"gopher"
["apple","peach","pear"]
{"apple":5,"lettuce":7}
{"Page":1,"Fruits":["apple","peach","pear"]}
{"page":1,"fruits":["apple","peach","pear"]}
map[num:6.13 strs:[a b]]
6.13
a
{1 [apple peach]}
apple
{"apple":5,"lettuce":7}

Program exited.
```

## ☁️ Outcome

✍️ I'm starting to get into some of the juicy stuff. Today, was kind of a long session. I'm motivated to complete the Go by example and start making Go programs

## Next Steps

✍️ Same. Start with XML and keep going.

## Social Proof

✍️ Show that you shared your process on Twitter or LinkedIn

[Toot](https://mastodon.social/@code_sentinel/111242298677501135)
