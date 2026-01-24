# Chapter 5: Error Handling in Go

If there's one aspect of Go that feels different from many other modern languages, it's error handling. Go eschews the popular `try-catch` exception model in favor of a more explicit, value-based approach. At first, this might seem verbose, but it leads to more robust and readable code.

## The `error` Type

In Go, errors are values. Specifically, they are values that satisfy the built-in `error` interface. It's a very simple interface:

```go
type error interface {
    Error() string
}
```

Any type that has a method named `Error` that returns a string is an `error`. This means you can create your own custom error types, but most of the time you'll use the `errors` package.

```go
import "errors"

// Create a new error value
err := errors.New("something went wrong")
fmt.Println(err.Error()) // prints "something went wrong"
```

## Idiomatic Error Handling with `if err != nil`

By convention, functions that can fail return an `error` as their last return value. The caller is expected to check this `error` value immediately. This leads to the most common Go idiom:

```go
import (
    "fmt"
    "strconv"
)

func main() {
    // strconv.Atoi can fail if the string is not a valid integer
    i, err := strconv.Atoi("42")
    if err != nil {
        // This is the error handling path
        fmt.Println("Could not convert:", err)
        return // or log, or exit, etc.
    }

    // This is the success path (the "happy path")
    fmt.Println("Converted integer:", i)
}
```

This pattern has several advantages:

1.  **Explicit:** It's impossible to ignore the fact that an error might occur. The `err` variable is right there.
2.  **Clear Control Flow:** The error handling logic is handled right next to the code that produced the error. The "happy path" of the code proceeds down the function, while errors cause an early return.
3.  **No Hidden Surprises:** Unlike exceptions, which can unwind the call stack from deep within a function, errors in Go are passed explicitly from one function to its caller.

Some newcomers find `if err != nil` repetitive, but experienced Go developers see it as a mark of clarity and reliability.

## Wrapping and Annotating Errors for Context

Sometimes, a simple error string isn't enough. You need to know *where* the error happened in the call stack. A common mistake is to just return the error you received from a function you called.

```go
// Bad practice: losing context
func readFile() error {
    f, err := os.Open("my-file.txt")
    if err != nil {
        return err // We don't know that the error came from os.Open
    }
    // ...
    return nil
}
```

A better approach is to add context to the error. Before Go 1.13, this was done with `fmt.Errorf` and the `%w` verb to "wrap" the original error.

```go
import (
    "fmt"
    "os"
)

func readFile() error {
    f, err := os.Open("my-file.txt")
    if err != nil {
        // Good practice: adding context
        return fmt.Errorf("failed to open file: %w", err)
    }
    // ...
    return nil
}
```

When you print this wrapped error, you'll see a chain: `failed to open file: open my-file.txt: no such file or directory`.

Go 1.13 introduced the `errors.Is` and `errors.As` functions to inspect wrapped errors.

*   `errors.Is(err, target)`: Checks if any error in the chain matches a specific error value (e.g., `io.EOF`).
*   `errors.As(err, &target)`: Checks if any error in the chain can be assigned to `target`, which is useful for custom error types.

```go
// Example of using errors.Is
_, err := io.Copy(dst, src)
if errors.Is(err, io.EOF) {
    // Not a "real" error, just the end of the stream
}
```

## Panic and Recover: When to Use (and When Not To)

Go has a `panic` function that causes the current goroutine to stop execution. After the `panic`, the program unwinds the call stack, running any `defer`red functions along the way. If the panic is not "recovered," the program crashes.

A `panic` is not for normal error handling. **You should not use `panic` for things like a file not being found or a failed network request.**

So, when *should* you use `panic`?

1.  **For truly exceptional situations:** A `panic` signifies a bug in your code, an unrecoverable state. For example, if a program is initialized with a configuration that is logically impossible, it might be appropriate to panic on startup. An out-of-bounds array access will also cause a panic.

2.  **Within a package, to be recovered:** Sometimes, `panic` and `recover` can be used to simplify error handling deep within a package, but the panic should not cross the package boundary. The public-facing API should still return an `error`. The standard library's `json` package is a good example of this pattern.

The `recover` function stops a panic and returns the value that was passed to `panic`. It's only useful inside a `defer`red function.

```go
func mightPanic() {
    defer func() {
        if r := recover(); r != nil {
            fmt.Println("Recovered from panic:", r)
        }
    }()

    fmt.Println("About to panic")
    panic("something bad happened")
    fmt.Println("This will never be printed")
}
```

**Rule of thumb:** As a beginner and for most of your Go career, you should focus on robust `error` handling. Avoid `panic` unless you have a very specific and well-justified reason. Proper `if err != nil` checking is the hallmark of a good Go programmer.
