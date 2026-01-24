# Chapter 10: Building Production-Ready APIs

Go is arguably one of the best languages for building APIs. Its performance, simplicity, and powerful standard library make it a natural fit for creating fast, reliable web services. In this chapter, we'll build on what we've learned to construct a production-ready RESTful API.

## The `net/http` Package for Building HTTP Servers

The foundation of any web service in Go is the `net/http` package. It provides all the tools you need to build an HTTP server. The core concepts are `http.Handler` and `http.ServeMux`.

An `http.Handler` is an interface with a single method: `ServeHTTP(ResponseWriter, *Request)`. Any type that implements this method can serve HTTP requests.

A `http.ServeMux` (or "mux") is an HTTP request multiplexer. It matches the URL of incoming requests against a list of registered patterns and calls the associated handler.

Here's a basic server:

```go
package main

import (
    "fmt"
    "log"
    "net/http"
)

// A simple handler function
func helloHandler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintln(w, "Hello, API!")
}

// A handler for a different route
func headersHandler(w http.ResponseWriter, r *http.Request) {
    for name, headers := range r.Header {
        for _, h := range headers {
            fmt.Fprintf(w, "%v: %v\n", name, h)
        }
    }
}

func main() {
    mux := http.NewServeMux()
    mux.HandleFunc("/hello", helloHandler)
    mux.HandleFunc("/headers", headersHandler)

    log.Println("Starting server on :8080")
    err := http.ListenAndServe(":8080", mux)
    log.Fatal(err) // This will only run if the server fails
}
```
This is a great start, but the standard library's mux is very basic. For a real application, you'll want more advanced features like path parameters and middleware.

## Routing and Middleware with `gorilla/mux` or `chi`

While you *can* build everything with the standard library, several excellent third-party routers provide more power and a more expressive API. Two of the most popular are `gorilla/mux` and `chi`. I'm a fan of `chi` for its simplicity and composability, but both are great choices.

Let's rewrite our server using `chi`.

First, get the package: `go get github.com/go-chi/chi/v5`

```go
package main

import (
    "log"
    "net/http"
    "github.com/go-chi/chi/v5"
    "github.com/go-chi/chi/v5/middleware"
)

func main() {
    r := chi.NewRouter()

    // Built-in middleware for logging, panic recovery, etc.
    r.Use(middleware.Logger)
    r.Use(middleware.Recoverer)

    r.Get("/", func(w http.ResponseWriter, r *http.Request) {
        w.Write([]byte("welcome"))
    })

    // A group of routes for /items
    r.Route("/items", func(r chi.Router) {
        r.Get("/", listItems)
        r.Post("/", createItem)
        r.Route("/{itemID}", func(r chi.Router) {
            r.Get("/", getItem) // GET /items/123
            r.Put("/", updateItem) // PUT /items/123
        })
    })

    log.Println("Starting server on :8080")
    http.ListenAndServe(":8080", r)
}
// Handler functions (listItems, createItem, etc.) would be defined elsewhere
```
Notice the use of `middleware`. Middleware is a powerful concept where handlers are chained together. A request passes through each middleware (which can read/write the request/response) before finally reaching your application handler. This is perfect for concerns like logging, authentication, compression, and panic recovery.

## Handling JSON: Encoding and Decoding

Modern APIs almost always communicate using JSON. Go's `encoding/json` package makes this straightforward.

-   **Decoding (Request Body)**: To read JSON from a request body into a Go struct, use `json.NewDecoder`.
-   **Encoding (Response Body)**: To write a Go struct as JSON to the response body, use `json.NewEncoder`.

Let's write a handler that creates an item.

```go
type Item struct {
    ID   string `json:"id"`
    Name string `json:"name"`
}

func createItem(w http.ResponseWriter, r *http.Request) {
    var newItem Item

    // Decode the request body into our struct
    err := json.NewDecoder(r.Body).Decode(&newItem)
    if err != nil {
        http.Error(w, err.Error(), http.StatusBadRequest)
        return
    }

    // In a real app, you would save the item to a database here
    // and assign it a real ID.
    log.Printf("Created item: %+v", newItem)
    newItem.ID = "some-unique-id"

    // Encode the new item as JSON and send it in the response
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(http.StatusCreated)
    json.NewEncoder(w).Encode(newItem)
}
```
The struct tags (e.g., `` `json:"name"` ``) control how the fields are mapped to and from JSON.

## Building a RESTful API with a Clean Architecture

Now let's put it all together. A "clean" architecture separates concerns. Your HTTP handlers should not contain business logic or database code.

1.  **Handlers (`/internal/api` or `/internal/transport/http`)**: Responsible for parsing requests, validating input, and calling the service layer. They handle the "HTTP" part of the problem.
2.  **Services (`/internal/service` or `/internal/domain`)**: Contains the core business logic. It doesn't know anything about HTTP. It defines interfaces for the data layer.
3.  **Repositories (`/internal/repository` or `/internal/database`)**: Responsible for data persistence. It implements the interfaces defined by the service layer.

Let's sketch out the structure for our `createItem` logic.

```go
// --- In /internal/database/item_repo.go ---
type ItemRepository struct {
    // db connection
}
func (r *ItemRepository) Create(item Item) (Item, error) {
    // SQL query to insert item
}

// --- In /internal/service/item_service.go ---
type ItemService struct {
    repo *database.ItemRepository
}
func (s *ItemService) CreateItem(name string) (Item, error) {
    // Business logic, e.g., validation
    if name == "" {
        return Item{}, errors.New("name cannot be empty")
    }
    newItem := Item{Name: name}
    return s.repo.Create(newItem)
}

// --- In /internal/api/handlers.go ---
type ItemHandler struct {
    service *service.ItemService
}
func (h *ItemHandler) CreateItem(w http.ResponseWriter, r *http.Request) {
    // Decode request body
    var req struct { Name string `json:"name"` }
    json.NewDecoder(r.Body).Decode(&req)
    
    // Call the service
    item, err := h.service.CreateItem(req.Name)
    if err != nil {
        // handle error and write HTTP response
        return
    }

    // Write success HTTP response
    json.NewEncoder(w).Encode(item)
}

// --- In /cmd/my-app/main.go ---
func main() {
    // 1. Initialize DB connection
    // 2. Create ItemRepository
    // 3. Create ItemService
    // 4. Create ItemHandler
    // 5. Set up router and pass it the handler
    // 6. Start server
}
```
This might seem like a lot of boilerplate, but this separation is crucial for building applications that are easy to test, maintain, and reason about. You can write unit tests for your services without needing an HTTP server or a database. You can swap out your database implementation without changing your business logic. This is the path to building professional, production-ready systems in Go.
