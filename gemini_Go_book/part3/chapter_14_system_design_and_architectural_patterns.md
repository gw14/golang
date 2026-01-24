# Chapter 14: System Design and Architectural Patterns

Writing code is one thing; designing systems is another. As you grow into a senior role, your responsibility shifts from implementing small features to designing the architecture of entire applications. Your decisions at this level will have a far greater impact on the success or failure of a project than the implementation of any single function.

Go's simplicity and its opinionated nature lend themselves to certain architectural patterns. Let's explore some of them.

## Principles of SOLID and Clean Architecture in Go

You may have heard of the SOLID principles in the context of object-oriented programming. While Go is not a traditional OO language, these principles are still highly relevant as they are fundamentally about building clean, maintainable systems.

-   **S**ingle Responsibility Principle: A package or a struct should have one, and only one, reason to change. We saw this in Chapter 10 when we separated our API into handlers, services, and repositories. Each had a single responsibility.
-   **O**pen/Closed Principle: Software entities should be open for extension, but closed for modification. In Go, this is the essence of interfaces. You can "extend" a function that accepts an interface by passing it a new type that satisfies the interface, without having to modify the original function.
-   **L**iskov Substitution Principle: Subtypes must be substitutable for their base types. Again, this is about interfaces. If a function expects a `Shape` interface, any type that implements `Shape` should work without causing the program to misbehave.
-   **I**nterface Segregation Principle: Don't force clients to depend on interfaces they don't use. Go encourages small, focused interfaces. Instead of a single, monolithic `ReadWriteCloser` interface, Go provides `io.Reader`, `io.Writer`, and `io.Closer`, allowing you to depend on only the behavior you need.
-   **D**ependency Inversion Principle: High-level modules should not depend on low-level modules. Both should depend on abstractions. This is the core of the clean architecture we discussed in Chapter 10. Our service layer depends on a repository *interface*, not on a concrete database implementation.

**Clean Architecture** in Go is all about this dependency inversion. You structure your application in layers (e.g., transport, service, repository), with dependencies always pointing inwards, towards the core business logic. The interfaces are defined in the inner layers and implemented by the outer layers. This creates a system that is decoupled, testable, and independent of external frameworks or databases.

## Common Design Patterns in Go

Many of the "Gang of Four" design patterns from the OO world have analogues in Go, though they often look different.

-   **Singleton**: In Go, this is often just a global variable in a package. To handle concurrent initialization safely, you use `sync.Once`.

    ```go
    var db *sql.DB
    var once sync.Once
    
    func GetDB() *sql.DB {
        once.Do(func() {
            // initialize db connection
            db = ...
        })
        return db
    }
    ```

-   **Factory**: A function that creates and returns an instance of a struct. This is common in Go, but we usually just call it a constructor function.

    ```go
    func NewItemService(repo ItemRepository) *ItemService {
        return &ItemService{repo: repo}
    }
    ```

-   **Decorator**: A decorator wraps an object to add new behavior. In Go, this is often done with function composition or by wrapping an interface. Middleware in `chi` or `net/http` is a perfect example of the decorator pattern.

    ```go
    func LoggingMiddleware(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            log.Println(r.Method, r.URL.Path)
            next.ServeHTTP(w, r)
        })
    }
    ```

Go's idioms favor composition over inheritance. You'll often find that complex patterns from other languages can be implemented in a much simpler way using Go's first-class functions and interfaces.

## Designing for Scalability and Resilience

-   **Scalability**: The ability of a system to handle a growing amount of work. In Go, this often means designing your application to be horizontally scalable. This means you can add more machines to your system to handle more load. A stateless API (where any instance of your service can handle any request) is the easiest to scale horizontally. State should be pushed out to a shared database or cache (like Redis).

-   **Resilience**: The ability of a system to tolerate failures. What happens if one of your database replicas goes down? What if a third-party API you depend on is slow?
    -   **Timeouts and Deadlines**: Use the `context` package for all I/O operations. Never make a network call without a timeout.
    -   **Retries**: If a network call fails, it might succeed if you try again. Implement a retry mechanism with an exponential backoff strategy to avoid overwhelming a struggling downstream service.
    -   **Circuit Breakers**: A circuit breaker is a component that wraps a function call. If the call fails repeatedly, the circuit breaker "trips" and for a period of time, all subsequent calls will fail immediately without even trying to execute the function. This prevents a failing service from cascading failures throughout your system. Libraries like `sony/gobreaker` provide this functionality.

## Trade-offs in System Design

There is no such thing as a perfect system. Every architectural decision is a trade-off. As a senior engineer, your job is to understand these trade-offs and make informed decisions.

-   **Consistency vs. Availability (CAP Theorem)**: In a distributed system, you can have at most two of the following three guarantees: Consistency (every read receives the most recent write), Availability (every request receives a response), and Partition Tolerance (the system continues to operate despite network partitions). You must choose which two are more important for your use case.
-   **Performance vs. Readability**: The highest-performance code is not always the most readable. A clever bit-twiddling hack might be fast, but if no one can understand it, it's a maintenance nightmare. Always write clear, simple code first. Only optimize after you have profiled and identified a real bottleneck.
-   **Coupling vs. Complexity**: A tightly coupled monolithic application is simple to deploy and understand at first. A decoupled microservices architecture is more resilient and scalable, but it introduces enormous operational complexity. The right choice depends on the size of your team, the maturity of your product, and your operational capabilities.

System design is not about finding the "right" answer. It's about understanding the constraints, weighing the options, and choosing the architecture that best fits the problem at hand. It's a skill learned through experience, by building things, and by seeing what works and what doesn't.
