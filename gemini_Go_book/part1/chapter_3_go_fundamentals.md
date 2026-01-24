# Chapter 3: Go Fundamentals

Now that your environment is set up, it's time to dive into the core syntax and concepts that form the backbone of every Go program. Think of these as your basic toolkit. Mastering them is essential before we can build anything substantial.

## Variables, Constants, and Basic Types

Go is a statically typed language, which means variable types are known at compile time. This helps catch many errors early.

### Variables

You declare variables using the `var` keyword, followed by the variable name and its type. Go initializes variables to their "zero value" if you don't provide an explicit one (e.g., `0` for numbers, `""` for strings, `false` for booleans, `nil` for pointers/slices/maps).

```go
var i int             // i is 0
var f float64         // f is 0.0
var b bool            // b is false
var s string          // s is ""
```

You can declare and initialize variables on the same line:

```go
var i int = 10
var s string = "Hello, Go!"
```

Go's type inference often allows for a shorter declaration using `:=` (the short declaration operator). This is common for local variables.

```go
i := 10             // type is int
s := "Hello, Go!"  // type is string
b := true           // type is bool
```
*Self-correction*: You can only use `:=` inside functions. Outside functions, you must use `var` or `const` with an explicit type or initial value.

### Constants

Constants are declared using the `const` keyword. They can be character, string, boolean, or numeric values. They cannot be declared using `:=`.

```go
const Pi = 3.14159
const Greeting = "Hello, Go Developers"
```

Go also has `iota`, which creates a sequence of incremental untyped integer constants. This is often used for enumerations.

```go
const (
    // iota is reset to 0 for each const block
    Sunday = iota // 0
    Monday        // 1
    Tuesday       // 2
)

const (
    _ = iota # blank identifier to discard 0
    KB = 1 << (10 * iota) # 1 << 10 = 1024
    MB = 1 << (10 * iota) # 1 << 20 = 1048576
    GB = 1 << (10 * iota) # 1 << 30
)
```

### Basic Types

Go's basic types include:

*   **Booleans:** `bool` (`true` or `false`)
*   **Numeric Types:**
    *   Integers: `int`, `int8`, `int16`, `int32`, `int64`, `uint`, `uint8`, `uint16`, `uint32`, `uint64`, `uintptr`. The plain `int` and `uint` are typically 32 or 64 bits depending on the system.
    *   Floating-point: `float32`, `float64`.
    *   Complex numbers: `complex64`, `complex128`.
*   **Strings:** `string` (immutable sequence of bytes, typically UTF-8 encoded).

## Composite Types: Arrays, Slices, Maps, and Structs

These types allow you to group multiple values together.

### Arrays

Arrays are fixed-size sequences of elements of the same type. Their size is part of their type.

```go
var a [5]int           // array of 5 integers, all 0
b := [3]int{1, 2, 3}   // array of 3 integers
c := [...]int{4, 5, 6} # compiler infers length 3
```
Arrays are fundamental but less common in everyday Go code than slices.

### Slices

Slices are a more powerful, flexible, and common alternative to arrays. They are dynamic-length sequences that reference a contiguous segment of an underlying array.

```go
var s []int            // nil slice
s = make([]int, 5)     // slice of 5 ints, capacity 5, all 0
t := []int{1, 2, 3, 4} # slice literal
u := t[1:3]            // creates a new slice referencing t[1] and t[2]
```
Key slice operations:
*   `len(s)`: Returns the length of the slice.
*   `cap(s)`: Returns the capacity of the underlying array.
*   `append(s, elements...)`: Appends elements to the slice. If capacity is exceeded, a new, larger underlying array is allocated.

### Maps

Maps are unordered collections of key-value pairs, where all keys are of one type and all values are of another.

```go
var m map[string]int     // nil map
m = make(map[string]int) # empty map, ready for use
m["apple"] = 1
m["banana"] = 2

# Map literal
ages := map[string]int{
    "Alice": 30,
    "Bob":   25,
}

val, ok := ages["Alice"] # Check for existence: ok will be true if key exists
delete(ages, "Bob")      # Delete a key-value pair
```

### Structs

Structs are user-defined composite types that group together fields of different types under a single name. They are Go's way of defining objects without classes.

```go
type Person struct {
    FirstName string
    LastName  string
    Age       int
}

p := Person{"Alice", "Smith", 30}
fmt.Println(p.FirstName) # Access fields with dot notation

# Anonymous struct (less common, for temporary use)
anon := struct {
    Name string
    ID   int
}{"Temp", 123}
```
Structs are key to organizing your data and will be used extensively throughout your Go journey.

## Control Flow: `if/else`, `for`, `switch`

Go's control flow statements are straightforward and lack parentheses around conditions.

### `if`/`else`

```go
if x > 10 {
    fmt.Println("x is greater than 10")
} else if x < 5 {
    fmt.Println("x is less than 5")
} else {
    fmt.Println("x is between 5 and 10 (inclusive)")
}

# Short statement: variable declared in 'if' scope
if val, err := someFunction(); err != nil {
    # handle error
} else {
    # use val
}
```

### `for` Loop

Go only has one looping construct: `for`. It's incredibly versatile.

```go
# Traditional C-style for loop
for i := 0; i < 10; i++ {
    fmt.Println(i)
}

# While loop equivalent
sum := 1
for sum < 100 {
    sum += sum
}

# Infinite loop
for {
    # do something
}

# Range over slices, arrays, maps, strings, channels
numbers := []int{1, 2, 3}
for index, value := range numbers {
    fmt.Printf("Index: %d, Value: %d\n", index, value)
}
```

### `switch` Statement

`switch` statements are more powerful than in many other languages; they don't require `break` statements.

```go
finger := 2
switch finger {
case 1:
    fmt.Println("Thumb")
case 2:
    fmt.Println("Index")
case 3, 4: # Multiple cases
    fmt.Println("Middle or Ring")
default:
    fmt.Println("Other finger")
}

# Type switch (for interfaces, covered later)
var i interface{} = "hello"
switch v := i.(type) {
case int:
    fmt.Println("Type is int")
case string:
    fmt.Println("Type is string:", v)
default:
    fmt.Println("Unknown type")
}

# Switch with no expression (acts like if/else if/else)
t := time.Now()
switch {
case t.Hour() < 12:
    fmt.Println("Good morning!")
case t.Hour() < 17:
    fmt.Println("Good afternoon.")
default:
    fmt.Println("Good evening.")
}
```

## Pointers: The What, Why, and How

Go has pointers, but they are much simpler and safer than in C/C++. You cannot perform pointer arithmetic. Pointers hold the memory address of a value.

*   `&` operator: Gives you the memory address of a variable (a pointer to it).
*   `*` operator: Dereferences a pointer, giving you the value stored at that memory address.

```go
x := 10
p := &x          # p is a pointer to x
fmt.Println(*p)  # prints 10 (the value x points to)
*p = 20          # changes the value at the address p points to
fmt.Println(x)   # x is now 20
```

### Why use pointers?

1.  **Modify a value in a function:** When you pass a variable to a function, Go typically passes a *copy* of the value. To modify the original value, you must pass a pointer.

    ```go
    func increment(val *int) {
        *val++ # Dereference and increment
    }

    func main() {
        num := 5
        increment(&num)
        fmt.Println(num) # prints 6
    }
    ```

2.  **Efficiency for large structs:** Passing large structs by value can be expensive as the entire struct is copied. Passing a pointer to the struct is more efficient, as only the memory address (typically 8 bytes) is copied.

3.  **Representing optional values:** A `nil` pointer can indicate the absence of a value, which is useful in certain scenarios.

Understanding these fundamentals is crucial. Go's simplicity means that these basic constructs are the workhorses of almost every Go program. The next chapter will build on this by showing you how to organize these fundamentals into reusable functions, methods, and the powerful concept of interfaces.
