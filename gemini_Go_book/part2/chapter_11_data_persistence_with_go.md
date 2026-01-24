# Chapter 11: Data Persistence with Go

Very few applications are stateless. Most need to store and retrieve data from a database. Go is not an object-oriented language, so its approach to data persistence can feel different from languages with feature-rich Object-Relational Mappers (ORMs). However, Go's standard library provides a solid foundation, and the ecosystem offers a range of tools to suit different needs.

## Working with SQL Databases using `database/sql`

The `database/sql` package is the standard, built-in way to work with SQL databases. It provides a generic SQL interface that can be used with any database that has a Go driver.

It's important to understand that `database/sql` itself is *not* a database driver. It's an abstraction layer. You'll need to import a specific driver for your database of choice (e.g., `github.com/lib/pq` for PostgreSQL, `github.com/go-sql-driver/mysql` for MySQL).

```go
import (
    "database/sql"
    "log"
    _ "github.com/lib/pq" // The blank import for the driver
)

type Item struct {
    ID   int
    Name string
}

func main() {
    // The connection string will vary depending on your database
    connStr := "user=user password=pass dbname=mydb sslmode=disable"
    db, err := sql.Open("postgres", connStr)
    if err != nil {
        log.Fatal(err)
    }
    defer db.Close()

    // It's a good practice to ping the database to ensure the connection is alive
    if err := db.Ping(); err != nil {
        log.Fatal(err)
    }

    // --- Querying for a single row ---
    var item Item
    row := db.QueryRow("SELECT id, name FROM items WHERE id = $1", 1)
    if err := row.Scan(&item.ID, &item.Name); err != nil {
        if err == sql.ErrNoRows {
            log.Println("No item found with that ID")
        } else {
            log.Fatal(err)
        }
    }
    fmt.Printf("Single item: %+v\n", item)

    // --- Querying for multiple rows ---
    rows, err := db.Query("SELECT id, name FROM items")
    if err != nil {
        log.Fatal(err)
    }
    defer rows.Close()

    var items []Item
    for rows.Next() {
        var it Item
        if err := rows.Scan(&it.ID, &it.Name); err != nil {
            log.Fatal(err)
        }
        items = append(items, it)
    }
    fmt.Printf("Multiple items: %+v\n", items)

    // --- Executing a statement (INSERT, UPDATE, DELETE) ---
    result, err := db.Exec("UPDATE items SET name = $1 WHERE id = $2", "New Name", 1)
    if err != nil {
        log.Fatal(err)
    }
    rowsAffected, _ := result.RowsAffected()
    fmt.Printf("%d rows affected\n", rowsAffected)
}
```
The `database/sql` package manages a connection pool for you. You `Open` the database once when your application starts and then share the `*sql.DB` object throughout your application.

## Using an ORM-like library (e.g., `sqlx`, `GORM`)

While `database/sql` is powerful, all that `row.Scan(...)` boilerplate can be tedious. This has led to the creation of libraries that build on top of the standard package to provide a more convenient API.

-   **`sqlx`**: My personal preference. `sqlx` is a minimal extension to `database/sql`. It provides functions that can scan query results directly into structs (if the column names match the struct field names), handle `IN` queries gracefully, and more. It gives you convenience without hiding the underlying SQL.

    ```go
    // With sqlx, the query part becomes much cleaner:
    var item Item
    err := db.Get(&item, "SELECT * FROM items WHERE id=$1", 1)
    
    var items []Item
    err = db.Select(&items, "SELECT * FROM items")
    ```

-   **`GORM`**: A full-featured ORM. GORM provides a much higher level of abstraction. You work with Go objects, and GORM translates your operations into SQL.

    ```go
    // With GORM, you might do something like this:
    var item Item
    db.First(&item, 1) // find item with integer primary key

    var items []Item
    db.Find(&items)
    ```

**My advice:** Start with `sqlx`. It reduces boilerplate while still requiring you to write and understand the SQL your application is running. Full ORMs like GORM can be convenient, but they can also be a leaky abstraction; when performance issues arise, you'll need to understand the SQL they generate anyway.

## Connecting to NoSQL Databases (e.g., MongoDB, Redis)

Go is not limited to SQL databases. Most popular NoSQL databases provide official or community-supported Go drivers. The approach is similar: you import the driver, connect to the database, and then use the driver's specific API to perform operations.

For example, with the official MongoDB driver:

```go
// import "go.mongodb.org/mongo-driver/mongo"
// import "go.mongodb.org/mongo-driver/mongo/options"

client, err := mongo.Connect(ctx, options.Client().ApplyURI("mongodb://localhost:27017"))
// ... error handling ...

collection := client.Database("testdb").Collection("items")

// Insert a document
res, err := collection.InsertOne(ctx, bson.M{"name": "pi", "value": 3.14159})
id := res.InsertedID

// Find a document
var result struct{ Value float64 }
err = collection.FindOne(ctx, bson.M{"_id": id}).Decode(&result)
```
Each NoSQL driver will have its own API, so you'll need to read its documentation.

## Transactions and Data Integrity

Any time you need to perform multiple database operations that must either all succeed or all fail together, you need a transaction. The `database/sql` package has first-class support for this.

```go
func transferMoney(db *sql.DB, fromID, toID int, amount float64) error {
    // Begin the transaction
    tx, err := db.Begin()
    if err != nil {
        return err
    }

    // Defer a rollback in case of a panic.
    // If the transaction succeeds, we'll commit it, and the rollback will be a no-op.
    defer tx.Rollback()

    // Operation 1: Debit the 'from' account
    _, err = tx.Exec("UPDATE accounts SET balance = balance - $1 WHERE id = $2", amount, fromID)
    if err != nil {
        return err // The rollback will be triggered by the defer
    }

    // Operation 2: Credit the 'to' account
    _, err = tx.Exec("UPDATE accounts SET balance = balance + $1 WHERE id = $2", amount, toID)
    if err != nil {
        return err // The rollback will be triggered by the defer
    }

    // If we get here, both operations were successful, so commit the transaction
    return tx.Commit()
}
```
This pattern ensures data integrity. If the program crashes after the debit but before the credit, the deferred `tx.Rollback()` (or the database itself) will ensure that the incomplete transaction is discarded, leaving your data in a consistent state.

Choosing the right database and the right tools to interact with it is a key architectural decision. Go's ecosystem provides a pragmatic set of options, from the low-level control of `database/sql` to the high-level convenience of ORMs, giving you the flexibility to make the right choice for your project.
