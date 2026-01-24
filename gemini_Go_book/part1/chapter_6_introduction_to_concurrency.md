# Chapter 6: Introduction to Concurrency

Concurrency is where Go truly shines. Many languages have libraries for concurrency, but Go has it built into the core language itself. This makes writing concurrent programs feel natural and simple. Go's philosophy is: "Do not communicate by sharing memory; instead, share memory by communicating."

This chapter provides a gentle introduction to the two main pillars of Go's concurrency model: goroutines and channels.

## Goroutines: Lightweight Threads of Execution

A *goroutine* is a lightweight thread of execution. It's not an OS thread; Go's runtime multiplexes multiple goroutines onto a smaller number of OS threads. This makes them incredibly cheap. You can have hundreds of thousands, or even millions, of goroutines running in a single program without breaking a sweat.

Starting a goroutine is simple: just put the `go` keyword before a function call.

```go
import (
    "fmt"
    "time"
)

func say(s string) {
    for i := 0; i < 3; i++ {
        fmt.Println(s)
        time.Sleep(100 * time.Millisecond)
    }
}

func main() {
    go say("World") // Start a new goroutine
    say("Hello")    // Run in the main goroutine
}
```

If you run this program, you'll see "Hello" and "World" interleaved. The `main` function runs in its own goroutine (the "main goroutine"). When you call `go say("World")`, a new goroutine is created that runs the `say` function concurrently with the `main` goroutine.

Notice that we don't have to wait for `say("World")` to finish. The `main` function continues and calls `say("Hello")`.

One subtlety: if the `main` goroutine finishes, the entire program exits, even if other goroutines are still running. If we had just done `go say("World")` and nothing else in `main`, the program would likely exit before "World" had a chance to print anything.

## Channels: Communicating Between Goroutines

Goroutines are great for doing things in parallel, but how do they talk to each other? The answer is *channels*. A channel is a typed conduit through which you can send and receive values between goroutines.

You create a channel using `make`:

```go
// Create a channel that can carry strings
ch := make(chan string)
```

The `<-` operator is used to send and receive values.

*   `ch <- value`: Send `value` to channel `ch`.
*   `value := <-ch`: Receive from `ch` and assign the value to `value`.

Let's modify our previous example to use a channel for communication.

```go
import (
    "fmt"
)

func greet(c chan string) {
    // Receive a value from the channel and print it
    greeting := <-c
    fmt.Println(greeting)
}

func main() {
    // Create a channel for strings
    ch := make(chan string)

    go greet(ch)

    // Send a value to the channel
    ch <- "Hello, Gopher!"

    // We need to wait here, otherwise main might exit too early.
    // In a real program, we'd use a more robust synchronization method.
    time.Sleep(1 * time.Second) 
}
```

By default, sends and receives on a channel are **blocking**. This is a crucial concept.

*   When you send a value to a channel, the send operation blocks until another goroutine is ready to receive the value.
*   When you receive a value from a channel, the receive operation blocks until another goroutine sends a value.

This blocking behavior is what allows goroutines to synchronize their execution without explicit locks or condition variables.

## The `select` Statement for Multi-Channel Operations

What if a goroutine needs to listen to multiple channels at once? The `select` statement lets a goroutine wait on several communication operations.

A `select` blocks until one of its cases can run, then it executes that case. If multiple cases are ready, it chooses one at random.

```go
import (
    "fmt"
    "time"
)

func main() {
    ch1 := make(chan string)
    ch2 := make(chan string)

    go func() {
        time.Sleep(2 * time.Second)
        ch1 <- "one"
    }()
    go func() {
        time.Sleep(1 * time.Second)
        ch2 <- "two"
    }()

    // We'll wait for both messages
    for i := 0; i < 2; i++ {
        select {
        case msg1 := <-ch1:
            fmt.Println("Received", msg1)
        case msg2 := <-ch2:
            fmt.Println("Received", msg2)
        }
    }
}
```
In this example, the `select` will first receive the message "two" from `ch2` after 1 second. Then, it will block again until it receives "one" from `ch1` a second later.

A `default` case in a `select` statement makes the operation non-blocking. If no other case is ready, the `default` case will run immediately.

## A Simple Concurrent Data Processor

Let's put it all together in a more realistic (though still simple) example: a worker pool. We'll have a set of jobs and a set of workers to process them.

```go
package main

import (
    "fmt"
    "time"
)

// The worker function. It receives jobs from the 'jobs' channel
// and sends results to the 'results' channel.
func worker(id int, jobs <-chan int, results chan<- int) {
    for j := range jobs {
        fmt.Printf("Worker %d started job %d\n", id, j)
        time.Sleep(time.Second) // Simulate work
        fmt.Printf("Worker %d finished job %d\n", id, j)
        results <- j * 2
    }
}

func main() {
    const numJobs = 5
    jobs := make(chan int, numJobs)
    results := make(chan int, numJobs)

    // Start up 3 workers
    for w := 1; w <= 3; w++ {
        go worker(w, jobs, results)
    }

    // Send 5 jobs to the 'jobs' channel
    for j := 1; j <= numJobs; j++ {
        jobs <- j
    }
    close(jobs) // Close the channel to signal that no more jobs will be sent

    // Collect the results of the work
    for a := 1; a <= numJobs; a++ {
        <-results
    }
}
```
This pattern is incredibly powerful. The `main` goroutine doesn't need to know which worker did which job. It just sends jobs and collects results. The channels handle all the synchronization.

This is just a taste of Go's concurrency model. We'll explore more advanced patterns in Part 2, but with just goroutines and channels, you can already build incredibly powerful and efficient concurrent programs.
