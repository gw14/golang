# Chapter 7: Project Structure and Design

You've mastered the fundamentals. You can write functions, use interfaces, and even spin up some goroutines. Now it's time to graduate from writing single-file scripts to building real, maintainable Go applications. How you structure your project is one of the most significant factors in its long-term success. A well-organized project is easy to navigate, test, and refactor. A messy one becomes a nightmare.

There's no single, universally enforced project structure in the Go community, but a strong consensus has emerged around a set of best practices. Let's walk through them.

## The Standard Go Project Layout

You'll often see a structure referred to as the "Standard Go Project Layout." It's important to understand that this is a *community convention*, not an official mandate from the Go team. You don't need all of these directories for every project, especially small ones. But it provides a fantastic starting point and a common vocabulary.

Here’s a look at some of the most important directories:

```
/
├── /cmd
│   └── /your-app-name
│       └── main.go
├── /internal
│   ├── /api
│   ├── /database
│   └── /...
├── /pkg
│   ├── /httpclient
│   └── /...
├── /api
├── go.mod
└── go.sum
```

- **/cmd**: This is where you put the `main` package for your application. If you have multiple executables, each gets its own subdirectory (e.g., `/cmd/my-app`, `/cmd/my-app-worker`). The code inside `/cmd` should be minimal; its main job is to parse arguments, set up configuration, and then call into the business logic, which lives elsewhere.

- **/internal**: This is for private application logic. Any code in the `/internal` directory can only be imported by code inside the same project (specifically, by code that shares the same parent directory as `/internal`). This is a compile-time guarantee enforced by the Go toolchain. It’s the perfect place for your core business logic, database access, and other code that you don't want other applications to import.

- **/pkg**: This is for library code that’s okay to be shared with and used by external applications. If you are writing a reusable library that other projects might import, this is where it would live. Be deliberate about what you put here. If you don't have a clear need to share a piece of code, it probably belongs in `/internal`.

- **/api**: If your application provides an API (e.g., OpenAPI/Swagger specs, gRPC `.proto` files), this directory is a good place to store the definition files.

## Designing Packages for Reusability and Clarity

Go's package system is its primary tool for organizing code. A package is a directory of Go files that are compiled together.

Here are my guiding principles for package design:

1.  **Name packages well.** Package names should be short, lowercase, and give a clear indication of their purpose. A package named `database` is better than `db_utils`. Avoid generic names like `util` or `common`; they tend to become a dumping ground for unrelated code.

2.  **Group by responsibility.** A package should have a single, clear responsibility. For example, in a web service, you might have a `user` package that handles user creation and authentication, and an `order` package that handles order processing. Don't group code by type (e.g., putting all your handlers in one package and all your models in another). Group by feature or domain concept.

3.  **The "dot import" is a code smell.** When you see `import . "mypackage"`, it's a sign that the package is not well-designed. It pulls all the public identifiers from `mypackage` into the current namespace, making the code harder to read.

## Managing Dependencies with Go Modules

We introduced Go Modules in Chapter 2, but let's reinforce their role in project structure. The `go.mod` file is the heart of your project's dependency management. It declares your module's path and lists its dependencies.

- **`go get`**: To add a new dependency, use `go get`. For example, `go get github.com/gorilla/mux` will download the package and add it to your `go.mod` and `go.sum` files.
- **`go mod tidy`**: This is your best friend. It cleans up the `go.mod` file, removing any unused dependencies and adding any that are missing. Run this before you commit your code.

A well-structured project keeps its `go.mod` file at the root of the project directory.

## Internal Packages: Encapsulating Your Logic

The `/internal` directory deserves special attention. It's one of the most powerful tools for creating a clean architecture.

By placing your core business logic in `/internal`, you are making a clear statement: "This is the heart of my application, and its implementation details are not a public API."

For example, your `/internal/database` package might contain the logic for connecting to and querying your database. Your `/cmd/my-app/main.go` file can import it, but another Go project on your machine cannot.

This forces you to think carefully about your public API. If you have code that *should* be shared, you can expose it through a clear, well-defined package in `/pkg`. But everything else should be hidden away in `/internal`.

This structure might seem like overkill for a simple "Hello, World!" app, but as your project grows, this discipline will pay for itself many times over. It keeps your code clean, prevents circular dependencies, and makes it much easier to reason about the boundaries of your application.
