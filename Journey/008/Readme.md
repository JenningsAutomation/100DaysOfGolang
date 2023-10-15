![gophers_in_school](schooltime2.png)

## Back To school

✍️ More Fundamentals. In this episode, I review Tickers, Worker Pools, WaitGroups, Rate Limiting, Atomic Counter and Mutexes

## Prerequisite

✍️ Have Golang installed

## Research

- ✍️ This is all from golang by example


### Step 1 — Tickers

Timers are for when you want to do something once in the future - tickers are for when you want to do something repeatedly at regular intervals. Here’s an example of a ticker that ticks periodically until we stop it.
```
// [Timers](timers) are for when you want to do
// something once in the future - _tickers_ are for when
// you want to do something repeatedly at regular
// intervals. Here's an example of a ticker that ticks
// periodically until we stop it.

package main

import (
	"fmt"
	"time"
)

func main() {

	// Tickers use a similar mechanism to timers: a
	// channel that is sent values. Here we'll use the
	// `select` builtin on the channel to await the
	// values as they arrive every 500ms.
	ticker := time.NewTicker(500 * time.Millisecond)
	done := make(chan bool)

	go func() {
		for {
			select {
			case <-done:
				return
			case t := <-ticker.C:
				fmt.Println("Tick at", t)
			}
		}
	}()

	// Tickers can be stopped like timers. Once a ticker
	// is stopped it won't receive any more values on its
	// channel. We'll stop ours after 1600ms.
	time.Sleep(1600 * time.Millisecond)
	ticker.Stop()
	done <- true
	fmt.Println("Ticker stopped")
}

```
outcome:
```
Tick at 2009-11-10 23:00:00.5 +0000 UTC m=+0.500000001
Tick at 2009-11-10 23:00:01 +0000 UTC m=+1.000000001
Tick at 2009-11-10 23:00:01.5 +0000 UTC m=+1.500000001
Ticker stopped

Program exited.
```

### Step 2 — Worker Pools

In this example we’ll look at how to implement a worker pool using goroutines and channels.

```
// In this example we'll look at how to implement
// a _worker pool_ using goroutines and channels.

package main

import (
	"fmt"
	"time"
)

// Here's the worker, of which we'll run several
// concurrent instances. These workers will receive
// work on the `jobs` channel and send the corresponding
// results on `results`. We'll sleep a second per job to
// simulate an expensive task.
func worker(id int, jobs <-chan int, results chan<- int) {
	for j := range jobs {
		fmt.Println("worker", id, "started  job", j)
		time.Sleep(time.Second)
		fmt.Println("worker", id, "finished job", j)
		results <- j * 2
	}
}

func main() {

	// In order to use our pool of workers we need to send
	// them work and collect their results. We make 2
	// channels for this.
	const numJobs = 5
	jobs := make(chan int, numJobs)
	results := make(chan int, numJobs)

	// This starts up 3 workers, initially blocked
	// because there are no jobs yet.
	for w := 1; w <= 3; w++ {
		go worker(w, jobs, results)
	}

	// Here we send 5 `jobs` and then `close` that
	// channel to indicate that's all the work we have.
	for j := 1; j <= numJobs; j++ {
		jobs <- j
	}
	close(jobs)

	// Finally we collect all the results of the work.
	// This also ensures that the worker goroutines have
	// finished. An alternative way to wait for multiple
	// goroutines is to use a [WaitGroup](waitgroups).
	for a := 1; a <= numJobs; a++ {
		<-results
	}
}

```
outcome:
```
worker 1 started  job 2
worker 3 started  job 1
worker 2 started  job 3
worker 2 finished job 3
worker 3 finished job 1
worker 3 started  job 5
worker 1 finished job 2
worker 2 started  job 4
worker 2 finished job 4
worker 3 finished job 5

Program exited.
```

### Step 3 — WaitGroups

To wait for multiple goroutines to finish, we can use a wait group.
```
// To wait for multiple goroutines to finish, we can
// use a *wait group*.

package main

import (
	"fmt"
	"sync"
	"time"
)

// This is the function we'll run in every goroutine.
func worker(id int) {
	fmt.Printf("Worker %d starting\n", id)

	// Sleep to simulate an expensive task.
	time.Sleep(time.Second)
	fmt.Printf("Worker %d done\n", id)
}

func main() {

	// This WaitGroup is used to wait for all the
	// goroutines launched here to finish. Note: if a WaitGroup is
	// explicitly passed into functions, it should be done *by pointer*.
	var wg sync.WaitGroup

	// Launch several goroutines and increment the WaitGroup
	// counter for each.
	for i := 1; i <= 5; i++ {
		wg.Add(1)
		// Avoid re-use of the same `i` value in each goroutine closure.
		// See [the FAQ](https://golang.org/doc/faq#closures_and_goroutines)
		// for more details.
		i := i

		// Wrap the worker call in a closure that makes sure to tell
		// the WaitGroup that this worker is done. This way the worker
		// itself does not have to be aware of the concurrency primitives
		// involved in its execution.
		go func() {
			defer wg.Done()
			worker(i)
		}()
	}

	// Block until the WaitGroup counter goes back to 0;
	// all the workers notified they're done.
	wg.Wait()

	// Note that this approach has no straightforward way
	// to propagate errors from workers. For more
	// advanced use cases, consider using the
	// [errgroup package](https://pkg.go.dev/golang.org/x/sync/errgroup).
}

```
outcome:
```
Worker 5 starting
Worker 1 starting
Worker 4 starting
Worker 2 starting
Worker 3 starting
Worker 3 done
Worker 5 done
Worker 2 done
Worker 1 done
Worker 4 done
```

### Step 4 — Rate Limiting
Rate limiting is an important mechanism for controlling resource utilization and maintaining quality of service. Go elegantly supports rate limiting with goroutines, channels, and tickers.
```
// [_Rate limiting_](https://en.wikipedia.org/wiki/Rate_limiting)
// is an important mechanism for controlling resource
// utilization and maintaining quality of service. Go
// elegantly supports rate limiting with goroutines,
// channels, and [tickers](tickers).

package main

import (
	"fmt"
	"time"
)

func main() {

	// First we'll look at basic rate limiting. Suppose
	// we want to limit our handling of incoming requests.
	// We'll serve these requests off a channel of the
	// same name.
	requests := make(chan int, 5)
	for i := 1; i <= 5; i++ {
		requests <- i
	}
	close(requests)

	// This `limiter` channel will receive a value
	// every 200 milliseconds. This is the regulator in
	// our rate limiting scheme.
	limiter := time.Tick(200 * time.Millisecond)

	// By blocking on a receive from the `limiter` channel
	// before serving each request, we limit ourselves to
	// 1 request every 200 milliseconds.
	for req := range requests {
		<-limiter
		fmt.Println("request", req, time.Now())
	}

	// We may want to allow short bursts of requests in
	// our rate limiting scheme while preserving the
	// overall rate limit. We can accomplish this by
	// buffering our limiter channel. This `burstyLimiter`
	// channel will allow bursts of up to 3 events.
	burstyLimiter := make(chan time.Time, 3)

	// Fill up the channel to represent allowed bursting.
	for i := 0; i < 3; i++ {
		burstyLimiter <- time.Now()
	}

	// Every 200 milliseconds we'll try to add a new
	// value to `burstyLimiter`, up to its limit of 3.
	go func() {
		for t := range time.Tick(200 * time.Millisecond) {
			burstyLimiter <- t
		}
	}()

	// Now simulate 5 more incoming requests. The first
	// 3 of these will benefit from the burst capability
	// of `burstyLimiter`.
	burstyRequests := make(chan int, 5)
	for i := 1; i <= 5; i++ {
		burstyRequests <- i
	}
	close(burstyRequests)
	for req := range burstyRequests {
		<-burstyLimiter
		fmt.Println("request", req, time.Now())
	}
}

```
outcome:
```
request 1 2009-11-10 23:00:00.2 +0000 UTC m=+0.200000001
request 2 2009-11-10 23:00:00.4 +0000 UTC m=+0.400000001
request 3 2009-11-10 23:00:00.6 +0000 UTC m=+0.600000001
request 4 2009-11-10 23:00:00.8 +0000 UTC m=+0.800000001
request 5 2009-11-10 23:00:01 +0000 UTC m=+1.000000001
request 1 2009-11-10 23:00:01 +0000 UTC m=+1.000000001
request 2 2009-11-10 23:00:01 +0000 UTC m=+1.000000001
request 3 2009-11-10 23:00:01 +0000 UTC m=+1.000000001
request 4 2009-11-10 23:00:01.2 +0000 UTC m=+1.200000001
request 5 2009-11-10 23:00:01.4 +0000 UTC m=+1.400000001

Program exited.
```
### Step 5 — Atomic Counters

The primary mechanism for managing state in Go is communication over channels. We saw this for example with worker pools. There are a few other options for managing state though. Here we’ll look at using the sync/atomic package for atomic counters accessed by multiple goroutines.
```
// The primary mechanism for managing state in Go is
// communication over channels. We saw this for example
// with [worker pools](worker-pools). There are a few other
// options for managing state though. Here we'll
// look at using the `sync/atomic` package for _atomic
// counters_ accessed by multiple goroutines.

package main

import (
	"fmt"
	"sync"
	"sync/atomic"
)

func main() {

	// We'll use an atomic integer type to represent our
	// (always-positive) counter.
	var ops atomic.Uint64

	// A WaitGroup will help us wait for all goroutines
	// to finish their work.
	var wg sync.WaitGroup

	// We'll start 50 goroutines that each increment the
	// counter exactly 1000 times.
	for i := 0; i < 50; i++ {
		wg.Add(1)

		go func() {
			for c := 0; c < 1000; c++ {

				// To atomically increment the counter we use `Add`.
				ops.Add(1)
			}

			wg.Done()
		}()
	}

	// Wait until all the goroutines are done.
	wg.Wait()

	// Here no goroutines are writing to 'ops', but using
	// `Load` it's safe to atomically read a value even while
	// other goroutines are (atomically) updating it.
	fmt.Println("ops:", ops.Load())
}

```
outcome:
```
ops: 50000

Program exited.
```

### Step 6 — Mutexes

In the previous example we saw how to manage simple counter state using atomic operations. For more complex state we can use a mutex to safely access data across multiple goroutines.
```
// In the previous example we saw how to manage simple
// counter state using [atomic operations](atomic-counters).
// For more complex state we can use a [_mutex_](https://en.wikipedia.org/wiki/Mutual_exclusion)
// to safely access data across multiple goroutines.

package main

import (
	"fmt"
	"sync"
)

// Container holds a map of counters; since we want to
// update it concurrently from multiple goroutines, we
// add a `Mutex` to synchronize access.
// Note that mutexes must not be copied, so if this
// `struct` is passed around, it should be done by
// pointer.
type Container struct {
	mu       sync.Mutex
	counters map[string]int
}

func (c *Container) inc(name string) {
	// Lock the mutex before accessing `counters`; unlock
	// it at the end of the function using a [defer](defer)
	// statement.
	c.mu.Lock()
	defer c.mu.Unlock()
	c.counters[name]++
}

func main() {
	c := Container{
		// Note that the zero value of a mutex is usable as-is, so no
		// initialization is required here.
		counters: map[string]int{"a": 0, "b": 0},
	}

	var wg sync.WaitGroup

	// This function increments a named counter
	// in a loop.
	doIncrement := func(name string, n int) {
		for i := 0; i < n; i++ {
			c.inc(name)
		}
		wg.Done()
	}

	// Run several goroutines concurrently; note
	// that they all access the same `Container`,
	// and two of them access the same counter.
	wg.Add(3)
	go doIncrement("a", 10000)
	go doIncrement("a", 10000)
	go doIncrement("b", 10000)

	// Wait for the goroutines to finish
	wg.Wait()
	fmt.Println(c.counters)
}

```
outcome:
```
map[a:20000 b:10000]

Program exited.
```

## ☁️ Outcome

✍️ Similar to other lessons, the real learning will come when I integrate this into my own projects

## Next Steps

✍️ More Go by Example

## Social Proof

✍️ [Toot](https://mastodon.social/@code_sentinel/111240652734277779)
