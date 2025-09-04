Here are five small programming tasks in Go:

### 1. Simple Command-Line To-Do List

Create a command-line application that allows a user to add, list, and remove to-do items. Store the to-do items in a slice of strings. When a user runs the program, they can use flags or arguments like `go run main.go add "Buy milk"`, `go run main.go list`, and `go run main.go remove 1` to interact with their list. This task introduces you to **command-line argument parsing**, working with **slices**, and basic **input/output**.

***

### 2. Simple HTTP Server

Build a basic HTTP server that listens on a port (e.g., 8080) and responds to a request. When a user navigates to `localhost:8080/hello`, the server should respond with a message like "Hello, Go!". You can also add another endpoint like `/time` that responds with the current time. This task is a great way to learn about Go's **`net/http` package** and **handling HTTP requests**.

***

### 3. File Word Counter

Write a program that takes a file path as a command-line argument and counts the frequency of each word in the file. The program should print a list of each word and its count. For example, if the file contains "hello world hello", the output should be "hello: 2, world: 1". This task teaches you about **file input/output**, working with **maps** to store key-value pairs, and **string manipulation**.

***

### 4. Basic Calculator

Develop a console application that takes two numbers and an operator (+, -, *, /) as command-line arguments and performs the calculation. For example, `go run main.go 10 + 5` should output `15`. Be sure to handle potential errors like division by zero or invalid operators. This task helps you practice **type conversion** (e.g., `strconv` package) and using a **`switch` statement** for control flow.

***

### 5. Goroutine and Channel Example

Create a program that uses **goroutines** and **channels**. Spawn a goroutine that generates a series of numbers (e.g., from 1 to 10) and sends them to a channel. The main function should receive these numbers from the channel and print them. This task is a fundamental exercise in understanding Go's **concurrency model**.
