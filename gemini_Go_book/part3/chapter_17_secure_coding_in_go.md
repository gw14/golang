# Chapter 17: Secure Coding in Go

Building a feature-rich, high-performance application is pointless if it's not secure. Security is not something you can sprinkle on at the end; it must be a core consideration throughout the development lifecycle. As a senior developer, you are on the front lines of defense against attackers. Go has some great properties that help with security (it's memory-safe, statically typed), but there are still many pitfalls to be aware of.

## Common Vulnerabilities and How to Avoid Them in Go

Many classic web vulnerabilities are still relevant in Go applications. Here's how to defend against them.

### SQL Injection

This occurs when user input is concatenated directly into a SQL query, allowing an attacker to execute arbitrary SQL.

```go
// VULNERABLE CODE - DO NOT USE
userID := r.URL.Query().Get("id")
rows, err := db.Query("SELECT * FROM users WHERE id = '" + userID + "'")
```
If an attacker provides a `userID` of `' OR 1=1; --`, the query becomes `SELECT * FROM users WHERE id = '' OR 1=1; --`, which would return all users.

**Defense**: Always use parameterized queries. The `database/sql` package and related libraries (`sqlx`, etc.) support this out of the box. The database driver will safely handle the user input.

```go
// SECURE CODE
userID := r.URL.Query().Get("id")
rows, err := db.Query("SELECT * FROM users WHERE id = $1", userID) // Use $1 for Postgres, ? for MySQL
```

### Cross-Site Scripting (XSS)

This occurs when an application includes untrusted user data in its HTML response without escaping it. An attacker can inject malicious `<script>` tags that will be executed in the browser of other users.

**Defense**: Use Go's `html/template` package, not `text/template`, for rendering HTML. The `html/template` package is context-aware and automatically escapes data to prevent XSS.

```go
// VULNERABLE with text/template
// If userInput is "<script>alert('pwned')</script>", the script will execute.
t, _ := text.Template.New("foo").Parse(`Hello, {{.}}!`)
t.Execute(w, userInput)

// SECURE with html/template
// The < and > characters will be escaped to &lt; and &gt;
t, _ := html.Template.New("foo").Parse(`Hello, {{.}}!`)
t.Execute(w, userInput)
```

### Cross-Site Request Forgery (CSRF)

An attacker tricks a logged-in user into making an unintentional request to your application (e.g., by having them click a malicious link or visit a page with a hidden form).

**Defense**: Use anti-CSRF tokens. Your server generates a unique, unpredictable token and includes it in forms. When the form is submitted, the server checks that the submitted token matches the one it expects for that user's session. Libraries like `gorilla/csrf` provide easy-to-use middleware for this.

## Managing Secrets and Configuration

Your application needs configuration, some of which is sensitive (database passwords, API keys). **Never, ever hard-code secrets in your source code.**

1.  **Read from the environment**: The simplest and most common approach is to store configuration in environment variables. This follows the "Twelve-Factor App" methodology. Libraries like `spf13/viper` can make it easy to read config from environment variables, config files, and other sources.

2.  **Use a secret management system**: For production environments, the best practice is to use a dedicated secret management tool like HashiCorp Vault, AWS Secrets Manager, or Google Cloud Secret Manager. Your application is given an identity, and it authenticates with the secret manager at startup to fetch the secrets it needs. This provides a central place to manage, rotate, and audit access to secrets.

## Authentication and Authorization Patterns

-   **Authentication (AuthN)**: Is this user who they say they are?
-   **Authorization (AuthZ)**: Is this user allowed to do what they are trying to do?

These two concepts are distinct and should be handled separately.

### Authentication

For stateful web applications, cookie-based sessions are a standard approach. The user logs in with a password, the server verifies it, and then gives the user a session cookie. For subsequent requests, the browser sends the cookie, and the server uses it to identify the user.

For stateless APIs (especially those consumed by single-page apps or other services), token-based authentication (like JWTs - JSON Web Tokens) is common.
1.  User authenticates (e.g., with username/password).
2.  Server issues a short-lived, signed JWT containing the user's ID and other claims.
3.  Client sends this JWT in the `Authorization` header for every API request.
4.  Server validates the JWT's signature and expiration time on every request.

### Authorization

Once a user is authenticated, you need to check if they have permission to perform an action. This logic belongs in your service layer or in a dedicated middleware.

-   **Role-Based Access Control (RBAC)**: The simplest form. Users are assigned roles (e.g., `admin`, `member`), and you check if the user's role has permission to access a resource.
    ```go
    func AdminOnlyMiddleware(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            user := userFromContext(r.Context())
            if user.Role != "admin" {
                http.Error(w, "Forbidden", http.StatusForbidden)
                return
            }
            next.ServeHTTP(w, r)
        })
    }
    ```
-   **More granular systems**: For complex applications, you might need more fine-grained permission systems (e.g., Attribute-Based Access Control or ReBAC). This often involves looking up permissions from a database or a dedicated authorization service.

## Writing Secure Go Web Applications

Here are some other general best practices:

-   **Use HTTPS**: Always use TLS for any production traffic. Go's `http.ListenAndServeTLS` makes this easy. In a modern deployment, you might terminate TLS at a load balancer, but the principle remains the same: encrypt traffic in transit.
-   **Limit dependencies**: Every third-party package you import is a potential security risk. Regularly audit your dependencies (`go mod graph`) and use tools like `govulncheck` to scan for known vulnerabilities in your dependency tree.
-   **Write small interfaces**: The principle of least privilege applies to your code too. If a function only needs to read from a request body, it should accept an `io.Reader`, not the entire `*http.Request`.
-   **Set timeouts**: As we've discussed before, always set timeouts for outbound requests and for your HTTP server itself (`http.Server.ReadTimeout`, `WriteTimeout`, `IdleTimeout`). This prevents slow-loris style denial-of-service attacks.

Security is a process, not a destination. It requires vigilance, a healthy sense of paranoia, and a commitment to following best practices.
