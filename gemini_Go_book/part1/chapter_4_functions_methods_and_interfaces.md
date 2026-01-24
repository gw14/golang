# Chapter 4: Functions, Methods, and Interfaces

With the fundamentals under our belt, we can now explore how to structure code in a clean, reusable, and composable way. In Go, this is achieved through functions, methods, and the language's powerful and unique take on interfaces.

## Defining and Calling Functions

Functions are the basic building blocks of a Go program. They bundle up a piece of logic that can be called from other parts of your code.

A function declaration includes the `func` keyword, the function name, a list of parameters, an optional list of return values, and the function body.

```go
// A simple function with no parameters or return values
func sayHello() {
    fmt.Println("Hello!")
}

// A function that takes two integers and returns their sum
func add(a int, b int) int {
    return a + b
}

// If multiple parameters have the same type, you can shorten it
func subtract(a, b int) int {
    return a - b
}
```

### Multiple Return Values

A key feature of Go is the ability for functions to return multiple values. This is most commonly used to return a result and an error value, as we'll see in the next chapter.

```go
// Returns a result and a boolean indicating success
func divide(a, b float64) (float64, bool) {
    if b == 0 {
        return 0, false // Cannot divide by zero
    }
    return a / b, true
}

// Call the function and handle the multiple return values
result, ok := divide(10.0, 2.0)
if ok {
    fmt.Println("Result:", result)
} else {
    fmt.Println("Division failed")
}
```

### Variadic Functions

Functions can accept a variable number of arguments by using the `...` syntax on the last parameter. Inside the function, this parameter is treated as a slice of its type.

```go
func sum(numbers ...int) int {
    total := 0
    for _, num := range numbers {
        total += num
    }
    return total
}

fmt.Println(sum(1, 2, 3))       // prints 6
fmt.Println(sum(10, 20, 30, 40)) // prints 100
```

## Methods and the Concept of Receivers

A method is a function with a special *receiver* argument. The receiver appears in its own argument list between the `func` keyword and the method name. This is how Go attaches behavior to its `struct` types.

Let's define a `Circle` struct and attach a method to it.

```go
type Circle struct {
    Radius float64
}

// This 'Area' method has a receiver of type Circle named 'c'
func (c Circle) Area() float64 {
    return math.Pi * c.Radius * c.Radius
}

// To call the method:
c := Circle{Radius: 5}
fmt.Println(c.Area())
```

### Value vs. Pointer Receivers

The receiver can be either a value or a pointer. The choice is important:

*   **Value Receiver (`func (c Circle) ...`):** The method operates on a *copy* of the `Circle` value. Any modifications made inside the method will not affect the original `Circle`. Use this when the method doesn't need to modify the receiver.

*   **Pointer Receiver (`func (c *Circle) ...`):** The method operates on a *pointer* to the original `Circle` value. This allows the method to modify the receiver. Use this when you need to change the state of the struct. It's also more efficient for large structs as it avoids copying the entire struct.

```go
// A method with a pointer receiver to modify the Circle
func (c *Circle) Scale(factor float64) {
    c.Radius *= factor
}

c := Circle{Radius: 5}
c.Scale(2)
fmt.Println(c.Radius) // prints 10
```

As a rule of thumb: if in doubt, use a pointer receiver.

## Interfaces: The Power of Implicit Contracts

Interfaces are one of Go's most powerful and distinctive features. An interface type is defined as a set of method signatures.

```go
// An interface for any type that can calculate its area
type Shape interface {
    Area() float64
}
```

A type *implicitly* satisfies an interface if it defines all the methods in the interface. There's no `implements` keyword. This is a core tenet of Go: "If it walks like a duck and quacks like a duck, it's a duck."

Our `Circle` type already has an `Area() float64` method, so it automatically satisfies the `Shape` interface. Let's add another type:

```go
type Rectangle struct {
    Width, Height float64
}

func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}
```

Now, `Rectangle` also satisfies the `Shape` interface. We can write functions that operate on `Shape`s, without caring about the concrete type (whether it's a `Circle` or a `Rectangle`).

```go
func printArea(s Shape) {
    fmt.Printf("The area of this shape is %0.2f\n", s.Area())
}

c := Circle{Radius: 5}
r := Rectangle{Width: 3, Height: 4}

printArea(c)
printArea(r)
```

This de-coupling of behavior from implementation is fundamental to writing flexible and maintainable Go code.

## Putting It Together: A Small Command-Line Application

Let's create a simple application that uses these concepts. We'll make a program that calculates the total cost of a list of items, some of which have a discount.

```go
package main

import "fmt"

// Item interface represents anything that can be priced
type Pricer interface {
    Price() float64
}

// Product is a standard item
type Product struct {
    Name  string
    Cost  float64
}

func (p Product) Price() float64 {
    return p.Cost
}

// DiscountedProduct has a discount applied
type DiscountedProduct struct {
    Product
    Discount float64 // e.g., 0.2 for 20% off
}

// It overrides the Price() method
func (d DiscountedProduct) Price() float64 {
    return d.Cost * (1 - d.Discount)
}

// calculateTotal takes a slice of Pricers and sums their prices
func calculateTotal(items ...Pricer) float64 {
    total := 0.0
    for _, item := range items {
        total += item.Price()
    }
    return total
}

func main() {
    book := Product{Name: "Go Book", Cost: 25.00}
    
    // Using an embedded struct for the discounted product
    pen := DiscountedProduct{
        Product: Product{Name: "Pen", Cost: 2.00},
        Discount: 0.1, // 10% off
    }

    total := calculateTotal(book, pen)
    fmt.Printf("Total cost: $%.2f\n", total)
}
```
This example shows how functions (`calculateTotal`), methods (`Price`), structs (`Product`, `DiscountedProduct`), and interfaces (`Pricer`) work together to create a clean, extensible system. We can add new types of items that satisfy the `Pricer` interface, and `calculateTotal` will work without any changes. This is the power of Go's composition-based design.

```