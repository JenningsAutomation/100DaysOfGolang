![school time](gophersinschool.png)

# Go by Example PArt IV

## Introduction

✍️ This is part IV of Go by Example. I cover:
- Channel Buffering
- Channel Synchronization
- Channel Directions
- Select
- Timeouts
- Non-Blocking Channel Operations
- Closing Channels
- Range over Channels
- Timers

## Prerequisite

✍️ Have golang installed

## Research

- ✍️ Everything is from [Go by Example](https://gobyexample.com/)


### Step 1 — Channel Buffering

By default channels are unbuffered, meaning that they will only accept sends (chan <-) if there is a corresponding receive (<- chan) ready to receive the sent value. Buffered channels accept a limited number of values without a corresponding receiver for those values.
```
// By default channels are _unbuffered_, meaning that they
// will only accept sends (`chan <-`) if there is a
// corresponding receive (`<- chan`) ready to receive the
// sent value. _Buffered channels_ accept a limited
// number of  values without a corresponding receiver for
// those values.

package main

import "fmt"

func main() {

	// Here we `make` a channel of strings buffering up to
	// 2 values.
	messages := make(chan string, 2)

	// Because this channel is buffered, we can send these
	// values into the channel without a corresponding
	// concurrent receive.
	messages <- "buffered"
	messages <- "channel"

	// Later we can receive these two values as usual.
	fmt.Println(<-messages)
	fmt.Println(<-messages)
}

```
output:
```
buffered
channel

Program exited.
```

### Step 2 — Channel Synchronization
We can use channels to synchronize execution across goroutines. Here’s an example of using a blocking receive to wait for a goroutine to finish. When waiting for multiple goroutines to finish, you may prefer to use a WaitGroup.
```
// We can use channels to synchronize execution
// across goroutines. Here's an example of using a
// blocking receive to wait for a goroutine to finish.
// When waiting for multiple goroutines to finish,
// you may prefer to use a [WaitGroup](waitgroups).

package main

import (
	"fmt"
	"time"
)

// This is the function we'll run in a goroutine. The
// `done` channel will be used to notify another
// goroutine that this function's work is done.
func worker(done chan bool) {
	fmt.Print("working...")
	time.Sleep(time.Second)
	fmt.Println("done")

	// Send a value to notify that we're done.
	done <- true
}

func main() {

	// Start a worker goroutine, giving it the channel to
	// notify on.
	done := make(chan bool, 1)
	go worker(done)

	// Block until we receive a notification from the
	// worker on the channel.
	<-done
}

```
output:
```
working...done

Program exited.
```

### Step 3 — Channel Directions
When using channels as function parameters, you can specify if a channel is meant to only send or receive values. This specificity increases the type-safety of the program.
```
// When using channels as function parameters, you can
// specify if a channel is meant to only send or receive
// values. This specificity increases the type-safety of
// the program.

package main

import "fmt"

// This `ping` function only accepts a channel for sending
// values. It would be a compile-time error to try to
// receive on this channel.
func ping(pings chan<- string, msg string) {
	pings <- msg
}

// The `pong` function accepts one channel for receives
// (`pings`) and a second for sends (`pongs`).
func pong(pings <-chan string, pongs chan<- string) {
	msg := <-pings
	pongs <- msg
}

func main() {
	pings := make(chan string, 1)
	pongs := make(chan string, 1)
	ping(pings, "passed message")
	pong(pings, pongs)
	fmt.Println(<-pongs)
}
```
output:
```
passed message

Program exited.
```

### Step 4 — Select

Go’s select lets you wait on multiple channel operations. Combining goroutines and channels with select is a powerful feature of Go.
```
// Go's _select_ lets you wait on multiple channel
// operations. Combining goroutines and channels with
// select is a powerful feature of Go.

package main

import (
	"fmt"
	"time"
)

func main() {

	// For our example we'll select across two channels.
	c1 := make(chan string)
	c2 := make(chan string)

	// Each channel will receive a value after some amount
	// of time, to simulate e.g. blocking RPC operations
	// executing in concurrent goroutines.
	go func() {
		time.Sleep(1 * time.Second)
		c1 <- "one"
	}()
	go func() {
		time.Sleep(2 * time.Second)
		c2 <- "two"
	}()

	// We'll use `select` to await both of these values
	// simultaneously, printing each one as it arrives.
	for i := 0; i < 2; i++ {
		select {
		case msg1 := <-c1:
			fmt.Println("received", msg1)
		case msg2 := <-c2:
			fmt.Println("received", msg2)
		}
	}
}

```
output:
```
received one
received two

Program exited.
```

### Step 5 — Timeouts
Timeouts are important for programs that connect to external resources or that otherwise need to bound execution time. Implementing timeouts in Go is easy and elegant thanks to channels and select.
```
// _Timeouts_ are important for programs that connect to
// external resources or that otherwise need to bound
// execution time. Implementing timeouts in Go is easy and
// elegant thanks to channels and `select`.

package main

import (
	"fmt"
	"time"
)

func main() {

	// For our example, suppose we're executing an external
	// call that returns its result on a channel `c1`
	// after 2s. Note that the channel is buffered, so the
	// send in the goroutine is nonblocking. This is a
	// common pattern to prevent goroutine leaks in case the
	// channel is never read.
	c1 := make(chan string, 1)
	go func() {
		time.Sleep(2 * time.Second)
		c1 <- "result 1"
	}()

	// Here's the `select` implementing a timeout.
	// `res := <-c1` awaits the result and `<-time.After`
	// awaits a value to be sent after the timeout of
	// 1s. Since `select` proceeds with the first
	// receive that's ready, we'll take the timeout case
	// if the operation takes more than the allowed 1s.
	select {
	case res := <-c1:
		fmt.Println(res)
	case <-time.After(1 * time.Second):
		fmt.Println("timeout 1")
	}

	// If we allow a longer timeout of 3s, then the receive
	// from `c2` will succeed and we'll print the result.
	c2 := make(chan string, 1)
	go func() {
		time.Sleep(2 * time.Second)
		c2 <- "result 2"
	}()
	select {
	case res := <-c2:
		fmt.Println(res)
	case <-time.After(3 * time.Second):
		fmt.Println("timeout 2")
	}
}

```
output:
```
timeout 1
result 2

Program exited.
```

### Step 6 — Non-Blocking Channel Operations
Basic sends and receives on channels are blocking. However, we can use select with a default clause to implement non-blocking sends, receives, and even non-blocking multi-way selects.
```
// Basic sends and receives on channels are blocking.
// However, we can use `select` with a `default` clause to
// implement _non-blocking_ sends, receives, and even
// non-blocking multi-way `select`s.

package main

import "fmt"

func main() {
	messages := make(chan string)
	signals := make(chan bool)

	// Here's a non-blocking receive. If a value is
	// available on `messages` then `select` will take
	// the `<-messages` `case` with that value. If not
	// it will immediately take the `default` case.
	select {
	case msg := <-messages:
		fmt.Println("received message", msg)
	default:
		fmt.Println("no message received")
	}

	// A non-blocking send works similarly. Here `msg`
	// cannot be sent to the `messages` channel, because
	// the channel has no buffer and there is no receiver.
	// Therefore the `default` case is selected.
	msg := "hi"
	select {
	case messages <- msg:
		fmt.Println("sent message", msg)
	default:
		fmt.Println("no message sent")
	}

	// We can use multiple `case`s above the `default`
	// clause to implement a multi-way non-blocking
	// select. Here we attempt non-blocking receives
	// on both `messages` and `signals`.
	select {
	case msg := <-messages:
		fmt.Println("received message", msg)
	case sig := <-signals:
		fmt.Println("received signal", sig)
	default:
		fmt.Println("no activity")
	}
}

```
output:
```
no message received
no message sent
no activity

Program exited.
```

### Step 7 — Closing Channels
Closing a channel indicates that no more values will be sent on it. This can be useful to communicate completion to the channel’s receivers.
```
// _Closing_ a channel indicates that no more values
// will be sent on it. This can be useful to communicate
// completion to the channel's receivers.

package main

import "fmt"

// In this example we'll use a `jobs` channel to
// communicate work to be done from the `main()` goroutine
// to a worker goroutine. When we have no more jobs for
// the worker we'll `close` the `jobs` channel.
func main() {
	jobs := make(chan int, 5)
	done := make(chan bool)

	// Here's the worker goroutine. It repeatedly receives
	// from `jobs` with `j, more := <-jobs`. In this
	// special 2-value form of receive, the `more` value
	// will be `false` if `jobs` has been `close`d and all
	// values in the channel have already been received.
	// We use this to notify on `done` when we've worked
	// all our jobs.
	go func() {
		for {
			j, more := <-jobs
			if more {
				fmt.Println("received job", j)
			} else {
				fmt.Println("received all jobs")
				done <- true
				return
			}
		}
	}()

	// This sends 3 jobs to the worker over the `jobs`
	// channel, then closes it.
	for j := 1; j <= 3; j++ {link
	// We await the worker using the
	// [synchronization](channel-synchronization) approach
	// we saw earlier.
	<-done
}

```
output:
```
sent job 1
sent job 2
sent job 3
sent all jobs
received job 1
received job 2
received job 3
received all jobs

Program exited.
```

### Step 8 — Range over Channels

In a previous example we saw how for and range provide iteration over basic data structures. We can also use this syntax to iterate over values received from a channel.
```
// In a [previous](range) example we saw how `for` and
// `range` provide iteration over basic data structures.
// We can also use this syntax to iterate over
// values received from a channel.

package main

import "fmt"

func main() {

	// We'll iterate over 2 values in the `queue` channel.
	queue := make(chan string, 2)
	queue <- "one"
	queue <- "two"
	close(queue)

	// This `range` iterates over each element as it's
	// received from `queue`. Because we `close`d the
	// channel above, the iteration terminates linkafter
	// receiving the 2 elements.
	for elem := range queue {
		fmt.Println(elem)
	}
}
```
output:
```
one
two

Program exited.
```

### Step 9 — Timers
We often want to execute Go code at some point in the future, or repeatedly at some interval. Go’s built-in timer and ticker features make both of these tasks easy. We’ll look first at timers and then at tickers.

```
// We often want to execute Go code at some point in the
// future, or repeatedly at some interval. Go's built-in
// _timer_ and _ticker_ features make both of these tasks
// easy. We'll look first at timers and then
// at [tickers](tickers).

package main

import (
	"fmt"
	"time"
)

func main() {

	// Timers represent a single event in the future. You
	// tell the timer how long you want to wait, and it
	// provides a channel that will be notified at that
	// time. This timer will wait 2 seconds.
	timer1 := time.NewTimer(2 * time.Second)

	// The `<-timer1.C` blocks on the timer's channel `C`
	// until it sends a value indicating that the timer
	// fired.
	<-timer1.C
	fmt.Println("Timer 1 fired")

	// If you just wanted to wait, you could have used
	// `time.Sleep`. One reason a timer may be useful is
	// that you can cancel the timer before it fires.
	// Here's an example of that.
	timer2 := time.NewTimer(time.Second)
	go func() {
		<-timer2.C
		fmt.Println("Timer 2 fired")
	}()
	stop2 := timer2.Stop()
	if stop2 {
		fmt.Println("Timer 2 stopped")
	}

	// Give the `timer2` enough time to fire, if it ever
	// was going to, to show it is in fact stopped.
	time.Sleep(2 * time.Second)
}

```
output:
```
$ go run timers.go
Timer 1 fired
Timer 2 stopped
```

## ☁️ Outcome

✍️ This is awesome material. To really absorb this, I am going to have review the material again and try it out in my own project multiple times

## Next Steps

✍️ Starting with tickers and as far as I can go.

## Social Proof

✍️ [Toot](https://mastodon.social/@code_sentinel/111204978322948725)
