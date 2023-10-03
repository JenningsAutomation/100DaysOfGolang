![BackToSchool](GopherInSchool.png)

# Sticking with the basics

## Introduction

✍️ In this session I continue with Go by example

## Prerequisite

✍️ Have golang installed and the ability to read


### Step 1 — String and Runes
A Go string is a read-only slice of bytes. The language and the standard library treat strings specially - as containers of text encoded in UTF-8. In other languages, strings are made of “characters”. In Go, the concept of a character is called a rune - it’s an integer that represents a Unicode code point. [This Go blog post](https://go.dev/blog/strings) is a good introduction to the topic.

```
// A Go string is a read-only slice of bytes. The language
// and the standard library treat strings specially - as
// containers of text encoded in [UTF-8](https://en.wikipedia.org/wiki/UTF-8).
// In other languages, strings are made of "characters".
// In Go, the concept of a character is called a `rune` - it's
// an integer that represents a Unicode code point.
// [This Go blog post](https://go.dev/blog/strings) is a good
// introduction to the topic.

package main

import (
	"fmt"
	"unicode/utf8"
)

func main() {

	// `s` is a `string` assigned a literal value
	// representing the word "hello" in the Thai
	// language. Go string literals are UTF-8
	// encoded text.
	const s = "สวัสดี"

	// Since strings are equivalent to `[]byte`, this
	// will produce the length of the raw bytes stored within.
	fmt.Println("Len:", len(s))

	// Indexing into a string produces the raw byte values at
	// each index. This loop generates the hex values of all
	// the bytes that constitute the code points in `s`.
	for i := 0; i < len(s); i++ {
		fmt.Printf("%x ", s[i])
	}
	fmt.Println()

	// To count how many _runes_ are in a string, we can use
	// the `utf8` package. Note that the run-time of
	// `RuneCountInString` depends on the size of the string,
	// because it has to decode each UTF-8 rune sequentially.
	// Some Thai characters are represented by multiple UTF-8
	// code points, so the result of this count may be surprising.
	fmt.Println("Rune count:", utf8.RuneCountInString(s))

	// A `range` loop handles strings specially and decodes
	// each `rune` along with its offset in the string.
	for idx, runeValue := range s {
		fmt.Printf("%#U starts at %d\n", runeValue, idx)
	}

	// We can achieve the same iteration by using the
	// `utf8.DecodeRuneInString` function explicitly.
	fmt.Println("\nUsing DecodeRuneInString")
	for i, w := 0, 0; i < len(s); i += w {
		runeValue, width := utf8.DecodeRuneInString(s[i:])
		fmt.Printf("%#U starts at %d\n", runeValue, i)
		w = width

		// This demonstrates passing a `rune` value to a function.
		examineRune(runeValue)
	}
}

func examineRune(r rune) {

	// Values enclosed in single quotes are _rune literals_. We
	// can compare a `rune` value to a rune literal directly.
	if r == 't' {
		fmt.Println("found tee")
	} else if r == 'ส' {
		fmt.Println("found so sua")
	}
}

```
Outside:
```

$ go run strings-and-runes.go
Len: 18
e0 b8 aa e0 b8 a7 e0 b8 b1 e0 b8 aa e0 b8 94 e0 b8 b5 
Rune count: 6
U+0E2A 'ส' starts at 0
U+0E27 'ว' starts at 3
U+0E31 'ั' starts at 6
U+0E2A 'ส' starts at 9
U+0E14 'ด' starts at 12
U+0E35 'ี' starts at 15
Using DecodeRuneInString
U+0E2A 'ส' starts at 0
found so sua
U+0E27 'ว' starts at 3
U+0E31 'ั' starts at 6
U+0E2A 'ส' starts at 9
found so sua
U+0E14 'ด' starts at 12
U+0E35 'ี' starts at 15
```


### Step 2 — Structs

Go’s structs are typed collections of fields. They’re useful for grouping data together to form records.
```
// Go's _structs_ are typed collections of fields.
// They're useful for grouping data together to form
// records.

package main

import "fmt"

// This `person` struct type has `name` and `age` fields.
type person struct {
	name string
	age  int
}

// `newPerson` constructs a new person struct with the given name.
func newPerson(name string) *person {
	// You can safely return a pointer to local variable
	// as a local variable will survive the scope of the function.
	p := person{name: name}
	p.age = 42
	return &p
}

func main() {

	// This syntax creates a new struct.
	fmt.Println(person{"Bob", 20})

	// You can name the fields when initializing a struct.
	fmt.Println(person{name: "Alice", age: 30})

	// Omitted fields will be zero-valued.
	fmt.Println(person{name: "Fred"})

	// An `&` prefix yields a pointer to the struct.
	fmt.Println(&person{name: "Ann", age: 40})

	// It's idiomatic to encapsulate new struct creation in constructor functions
	fmt.Println(newPerson("Jon"))

	// Access struct fields with a dot.
	s := person{name: "Sean", age: 50}
	fmt.Println(s.name)

	// You can also use dots with struct pointers - the
	// pointers are automatically dereferenced.
	sp := &s
	fmt.Println(sp.age)

	// Structs are mutable.
	sp.age = 51
	fmt.Println(sp.age)

	// If a struct type is only used for a single value, we don't
	// have to give it a name. The value can have an anonymous
	// struct type. This technique is commonly used for
	// [table-driven tests](testing-and-benchmarking).
	dog := struct {
		name   string
		isGood bool
	}{
		"Rex",
		true,
	}
	fmt.Println(dog)
}
```
Outcome:
```
$ go run structs.go
{Bob 20}
{Alice 30}
{Fred 0}
&{Ann 40}
&{Jon 42}
Sean
50
51
{Rex true}
```

### Step 4 — Methods

Go supports methods defined on struct types.
```
// Go supports _methods_ defined on struct types.

package main

import "fmt"

type rect struct {
	width, height int
}

// This `area` method has a _receiver type_ of `*rect`.
func (r *rect) area() int {
	return r.width * r.height
}

// Methods can be defined for either pointer or value
// receiver types. Here's an example of a value receiver.
func (r rect) perim() int {
	return 2*r.width + 2*r.height
}

func main() {
	r := rect{width: 10, height: 5}

	// Here we call the 2 methods defined for our struct.
	fmt.Println("area: ", r.area())
	fmt.Println("perim:", r.perim())

	// Go automatically handles conversion between values
	// and pointers for method calls. You may want to use
	// a pointer receiver type to avoid copying on method
	// calls or to allow the method to mutate the
	// receiving struct.
	rp := &r
	fmt.Println("area: ", rp.area())
	fmt.Println("perim:", rp.perim())
}
```
output:
```
$ go run methods.go 
area:  50
perim: 30
area:  50
perim: 30
```
### Step 5 - Interfaces
Interfaces are named collections of method signatures.
```
// _Interfaces_ are named collections of method
// signatures.

package main

import (
	"fmt"
	"math"
)

// Here's a basic interface for geometric shapes.
type geometry interface {
	area() float64
	perim() float64
}

// For our example we'll implement this interface on
// `rect` and `circle` types.
type rect struct {
	width, height float64
}
type circle struct {
	radius float64
}

// To implement an interface in Go, we just need to
// implement all the methods in the interface. Here we
// implement `geometry` on `rect`s.
func (r rect) area() float64 {
	return r.width * r.height
}
func (r rect) perim() float64 {
	return 2*r.width + 2*r.height
}

// The implementation for `circle`s.
func (c circle) area() float64 {
	return math.Pi * c.radius * c.radius
}
func (c circle) perim() float64 {
	return 2 * math.Pi * c.radius
}

// If a variable has an interface type, then we can call
// methods that are in the named interface. Here's a
// generic `measure` function taking advantage of this
// to work on any `geometry`.
func measure(g geometry) {
	fmt.Println(g)
	fmt.Println(g.area())
	fmt.Println(g.perim())
}

func main() {
	r := rect{width: 3, height: 4}
	c := circle{radius: 5}

	// The `circle` and `rect` struct types both
	// implement the `geometry` interface so we can use
	// instances of
	// these structs as arguments to `measure`.
	measure(r)
	measure(c)
}
```

output:
```
$ go run interfaces.go
{3 4}
12
14
{5}
78.53981633974483
31.41592653589793
```
### Step 6 - Struct Embedding

Go supports embedding of structs and interfaces to express a more seamless composition of types. This is not to be confused with //go:embed which is a go directive introduced in Go version 1.16+ to embed files and folders into the application binary.

```
// Go supports _embedding_ of structs and interfaces
// to express a more seamless _composition_ of types.
// This is not to be confused with [`//go:embed`](embed-directive) which is
// a go directive introduced in Go version 1.16+ to embed
// files and folders into the application binary.

package main

import "fmt"

type base struct {
	num int
}

func (b base) describe() string {
	return fmt.Sprintf("base with num=%v", b.num)
}

// A `container` _embeds_ a `base`. An embedding looks
// like a field without a name.
type container struct {
	base
	str string
}

func main() {

	// When creating structs with literals, we have to
	// initialize the embedding explicitly; here the
	// embedded type serves as the field name.
	co := container{
		base: base{
			num: 1,
		},
		str: "some name",
	}

	// We can access the base's fields directly on `co`,
	// e.g. `co.num`.
	fmt.Printf("co={num: %v, str: %v}\n", co.num, co.str)

	// Alternatively, we can spell out the full path using
	// the embedded type name.
	fmt.Println("also num:", co.base.num)

	// Since `container` embeds `base`, the methods of
	// `base` also become methods of a `container`. Here
	// we invoke a method that was embedded from `base`
	// directly on `co`.
	fmt.Println("describe:", co.describe())

	type describer interface {
		describe() string
	}

	// Embedding structs with methods may be used to bestow
	// interface implementations onto other structs. Here
	// we see that a `container` now implements the
	// `describer` interface because it embeds `base`.
	var d describer = co
	fmt.Println("describer:", d.describe())
}

```
outcome:
```
co={num: 1, str: some name}
also num: 1
describe: base with num=1
describer: base with num=1
```
### Step 7 - Generics

```
// Starting with version 1.18, Go has added support for
// _generics_, also known as _type parameters_.

package main

import "fmt"

// As an example of a generic function, `MapKeys` takes
// a map of any type and returns a slice of its keys.
// This function has two type parameters - `K` and `V`;
// `K` has the `comparable` _constraint_, meaning that
// we can compare values of this type with the `==` and
// `!=` operators. This is required for map keys in Go.
// `V` has the `any` constraint, meaning that it's not
// restricted in any way (`any` is an alias for `interface{}`).
func MapKeys[K comparable, V any](m map[K]V) []K {
	r := make([]K, 0, len(m))
	for k := range m {
		r = append(r, k)
	}
	return r
}

// As an example of a generic type, `List` is a
// singly-linked list with values of any type.
type List[T any] struct {
	head, tail *element[T]
}

type element[T any] struct {
	next *element[T]
	val  T
}

// We can define methods on generic types just like we
// do on regular types, but we have to keep the type
// parameters in place. The type is `List[T]`, not `List`.
func (lst *List[T]) Push(v T) {
	if lst.tail == nil {
		lst.head = &element[T]{val: v}
		lst.tail = lst.head
	} else {
		lst.tail.next = &element[T]{val: v}
		lst.tail = lst.tail.next
	}
}

func (lst *List[T]) GetAll() []T {
	var elems []T
	for e := lst.head; e != nil; e = e.next {
		elems = append(elems, e.val)
	}
	return elems
}

func main() {
	var m = map[int]string{1: "2", 2: "4", 4: "8"}

	// When invoking generic functions, we can often rely
	// on _type inference_. Note that we don't have to
	// specify the types for `K` and `V` when
	// calling `MapKeys` - the compiler infers them
	// automatically.
	fmt.Println("keys:", MapKeys(m))

	// ... though we could also specify them explicitly.
	_ = MapKeys[int, string](m)

	lst := List[int]{}
	lst.Push(10)
	lst.Push(13)
	lst.Push(23)
	fmt.Println("list:", lst.GetAll())
}

```
Outcome: 
```
keys: [1 2 4]
list: [10 13 23]

Program exited.
```

### Step 8 - Errors

In Go it’s idiomatic to communicate errors via an explicit, separate return value. This contrasts with the exceptions used in languages like Java and Ruby and the overloaded single result / error value sometimes used in C. Go’s approach makes it easy to see which functions return errors and to handle them using the same language constructs employed for any other, non-error tasks.
```
// In Go it's idiomatic to communicate errors via an
// explicit, separate return value. This contrasts with
// the exceptions used in languages like Java and Ruby and
// the overloaded single result / error value sometimes
// used in C. Go's approach makes it easy to see which
// functions return errors and to handle them using the
// same language constructs employed for any other,
// non-error tasks.

package main

import (
	"errors"
	"fmt"
)

// By convention, errors are the last return value and
// have type `error`, a built-in interface.
func f1(arg int) (int, error) {
	if arg == 42 {

		// `errors.New` constructs a basic `error` value
		// with the given error message.
		return -1, errors.New("can't work with 42")

	}

	// A `nil` value in the error position indicates that
	// there was no error.
	return arg + 3, nil
}

// It's possible to use custom types as `error`s by
// implementing the `Error()` method on them. Here's a
// variant on the example above that uses a custom type
// to explicitly represent an argument error.
type argError struct {
	arg  int
	prob string
}

func (e *argError) Error() string {
	return fmt.Sprintf("%d - %s", e.arg, e.prob)
}

func f2(arg int) (int, error) {
	if arg == 42 {

		// In this case we use `&argError` syntax to build
		// a new struct, supplying values for the two
		// fields `arg` and `prob`.
		return -1, &argError{arg, "can't work with it"}
	}
	return arg + 3, nil
}

func main() {

	// The two loops below test out each of our
	// error-returning functions. Note that the use of an
	// inline error check on the `if` line is a common
	// idiom in Go code.
	for _, i := range []int{7, 42} {
		if r, e := f1(i); e != nil {
			fmt.Println("f1 failed:", e)
		} else {
			fmt.Println("f1 worked:", r)
		}
	}
	for _, i := range []int{7, 42} {
		if r, e := f2(i); e != nil {
			fmt.Println("f2 failed:", e)
		} else {
			fmt.Println("f2 worked:", r)
		}
	}

	// If you want to programmatically use the data in
	// a custom error, you'll need to get the error as an
	// instance of the custom error type via type
	// assertion.
	_, e := f2(42)
	if ae, ok := e.(*argError); ok {
		fmt.Println(ae.arg)
		fmt.Println(ae.prob)
	}
}

```
Outcome:
```
f1 worked: 10
f1 failed: can't work with 42
f2 worked: 10
f2 failed: 42 - can't work with it
42
can't work with it

Program exited.
```
### Step 9 - Goroutines
A goroutine is a lightweight thread of execution.
```
// A _goroutine_ is a lightweight thread of execution.

package main

import (
	"fmt"
	"time"
)

func f(from string) {
	for i := 0; i < 3; i++ {
		fmt.Println(from, ":", i)
	}
}

func main() {

	// Suppose we have a function call `f(s)`. Here's how
	// we'd call that in the usual way, running it
	// synchronously.
	f("direct")

	// To invoke this function in a goroutine, use
	// `go f(s)`. This new goroutine will execute
	// concurrently with the calling one.
	go f("goroutine")

	// You can also start a goroutine for an anonymous
	// function call.
	go func(msg string) {
		fmt.Println(msg)
	}("going")

	// Our two function calls are running asynchronously in
	// separate goroutines now. Wait for them to finish
	// (for a more robust approach, use a [WaitGroup](waitgroups)).
	time.Sleep(time.Second)
	fmt.Println("done")
}

```
Outcome:
```
direct : 0
direct : 1
direct : 2
goroutine : 0
goroutine : 1
goroutine : 2
going
done
```
### Step 10 - Channels

Channels are the pipes that connect concurrent goroutines. You can send values into channels from one goroutine and receive those values into another goroutine.
```
// _Channels_ are the pipes that connect concurrent
// goroutines. You can send values into channels from one
// goroutine and receive those values into another
// goroutine.

package main

import "fmt"

func main() {

	// Create a new channel with `make(chan val-type)`.
	// Channels are typed by the values they convey.
	messages := make(chan string)

	// _Send_ a value into a channel using the `channel <-`
	// syntax. Here we send `"ping"`  to the `messages`
	// channel we made above, from a new goroutine.
	go func() { messages <- "ping" }()

	// The `<-channel` syntax _receives_ a value from the
	// channel. Here we'll receive the `"ping"` message
	// we sent above and print it out.
	msg := <-messages
	fmt.Println(msg)
}

```
Outcome:
```
ping

Program exited.
```



## ☁️ Outcome

✍️ I reviewed Strings and runes, structs, methods, interfaces, Struct embedding, generics, errors, goroutines and channels. It will take time to absorb all of this. I will have to spend some time putting them in action.

## Next Steps

✍️ Will continue with channel buffering and go from there.

## Social Proof

[Toot](https://mastodon.social/@code_sentinel/111172151147717540)
