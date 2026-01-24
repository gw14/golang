# Chapter 9: Comprehensive Testing in Go

If you want to build professional-grade software, you have to write tests. It's not optional. In my experience, a project's long-term health is more correlated with its test quality than with any other single factor. Go has a fantastic built-in testing framework and a strong culture of testing.

## The `testing` Package: Unit Tests and Benchmarks

The core of testing in Go is the `testing` package. Test files are easy to recognize: they are named `_test.go` and live in the same package as the code they are testing.

### Unit Tests

A test function is a function in a `_test.go` file that starts with `Test`. It takes one argument: `t *testing.T`.

Let's say we have a simple `add.go` file:

```go
// in add.go
package math

func Add(a, b int) int {
    return a + b
}
```

The test file, `add_test.go`, would look like this:

```go
// in add_test.go
package math

import "testing"

func TestAdd(t *testing.T) {
    got := Add(2, 2)
    want := 4

    if got != want {
        t.Errorf("Add(2, 2) = %d; want %d", got, want)
    }
}
```

To run the tests for the current package, you simply run `go test`. The tool will find all the `_test.go` files, compile and run them, and report the results.

The `*testing.T` type provides methods for reporting test failures. `t.Errorf` formats a string and marks the test as failed, but continues execution. `t.Fatalf` does the same but stops the test immediately.

### Benchmarks

The `testing` package also has built-in support for benchmarks. A benchmark function starts with `Benchmark` and takes one argument: `b *testing.B`. The Go toolchain will run the function multiple times until it has a stable measurement.

```go
// in add_test.go
func BenchmarkAdd(b *testing.B) {
    // b.N is the number of iterations the benchmark will run
    for i := 0; i < b.N; i++ {
        Add(100, 200)
    }
}
```

To run the benchmarks, use the `-bench` flag: `go test -bench=.`.

## Table-Driven Tests for Thoroughness

Writing a separate test function for every single case is tedious. A common and powerful pattern in Go is the "table-driven test." You define a slice of test cases (the "table"), and then iterate over them in your test function.

This makes it easy to add new test cases and keeps your test logic concise.

```go
func TestAddTable(t *testing.T) {
    testCases := []struct {
        name string
        a, b int
        want int
    }{
        {"positive numbers", 2, 3, 5},
        {"negative numbers", -2, -3, -5},
        {"mixed numbers", 2, -3, -1},
        {"zero", 0, 0, 0},
    }

    for _, tc := range testCases {
        t.Run(tc.name, func(t *testing.T) {
            got := Add(tc.a, tc.b)
            if got != tc.want {
                t.Errorf("got %d, want %d", got, tc.want)
            }
        })
    }
}
```
The `t.Run` function creates a subtest. This is great for organization and for getting clear output when a specific case fails.

## Test Doubles: Mocks and Stubs

Your code doesn't live in a vacuum. It has dependencies: databases, network services, file systems, etc. When you're writing a *unit test*, you want to test your code in isolation, without its real dependencies. This is where test doubles come in.

The key is to use interfaces. If your code depends on an interface, you can provide a "fake" implementation of that interface during testing.

-   **Stub**: A simple fake implementation that returns hard-coded values.
-   **Mock**: A more complex fake that you can set expectations on. For example, you can assert that a method was called exactly once with specific arguments.

Let's imagine a `Notifier` that sends notifications.

```go
type Notifier interface {
    Notify(message string) error
}

// Your real implementation might send an email or a Slack message.
// For testing, we can create a mock.

type MockNotifier struct {
    // Use fields to track calls and control behavior
    NotifyCalls int
    ShouldError bool
}

func (m *MockNotifier) Notify(message string) error {
    m.NotifyCalls++
    if m.ShouldError {
        return errors.New("failed to notify")
    }
    return nil
}

func TestSomethingThatNotifies(t *testing.T) {
    mock := &MockNotifier{}
    
    // Pass the mock to the function you're testing
    myFunctionThatUsesNotifier(mock)

    if mock.NotifyCalls != 1 {
        t.Errorf("expected Notify to be called once, got %d", mock.NotifyCalls)
    }
}
```
While you can write your own mocks, there are also popular libraries like `testify/mock` and `gomock` that can generate them for you.

## Integration Testing with `httptest` and Database Fixtures

Unit tests are essential, but they aren't enough. You also need *integration tests* to verify that the different parts of your system work together correctly.

### `httptest`

The standard library's `net/http/httptest` package is a gem for testing HTTP servers. It lets you create a mock `http.Request` and a mock `http.ResponseWriter` to pass to your `http.Handler` functions directly, without needing to spin up a real server.

```go
func TestMyAPIHandler(t *testing.T) {
    req := httptest.NewRequest("GET", "http://example.com/foo", nil)
    w := httptest.NewRecorder()

    myAPIHandler(w, req)

    resp := w.Result()
    body, _ := io.ReadAll(resp.Body)

    if resp.StatusCode != http.StatusOK {
        t.Errorf("expected status OK, got %v", resp.Status)
    }
    // ... check body content ...
}
```

For end-to-end tests, `httptest.NewServer` can even create a real, running server on a local port that your tests can make actual HTTP requests against.

### Database Fixtures

Testing code that interacts with a database is a classic integration testing challenge. The best approach is often to run your tests against a real, but temporary, database.

1.  **Use a containerized database**: In your CI pipeline (and on your local machine), spin up a Docker container with a real database (e.g., Postgres, MySQL).
2.  **Isolate tests**: Each test should run in its own transaction that is rolled back at the end of the test. This ensures that tests don't interfere with each other.
3.  **Load test data**: Before each test (or test suite), load a known set of "fixture" data into the database so your test can make assertions against a predictable state.

This requires more setup, but it gives you the highest confidence that your data access logic is correct.

A comprehensive test suite with a mix of fast unit tests and thorough integration tests is the bedrock of a healthy Go project. It enables you to refactor with confidence and catch bugs before they reach production.
