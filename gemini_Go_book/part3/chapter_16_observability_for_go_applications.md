# Chapter 16: Observability for Go Applications

In a monolithic world, you could often debug a problem by SSH-ing into a server and looking at a log file. In the distributed world of microservices, that's not feasible. Your request might be handled by one of dozens of identical service instances, and it might trigger a chain of calls to other services. To understand what's happening inside such a system, you need **observability**.

Observability is more than just monitoring. It's about being able to ask arbitrary questions about your system from the outside, without having to ship new code to answer them. It's typically understood as having three pillars: **logs**, **metrics**, and **traces**.

## Structured Logging with `slog`

We've been using the standard `log` package throughout this book. It's fine for simple applications, but for a real system, you need structured logs. Instead of printing plain text, structured logging formats logs as JSON or another machine-readable format. This allows you to easily search, filter, and analyze your logs in a centralized logging platform (like Splunk, Datadog, or an ELK stack).

As of Go 1.21, Go has a fantastic structured logging package in the standard library: `log/slog`.

```go
import (
    "log/slog"
    "os"
)

func main() {
    // Create a new JSON logger writing to standard out
    logger := slog.New(slog.NewJSONHandler(os.Stdout, nil))

    // Log a simple message
    logger.Info("Hello, structured logs!")

    // Log with additional, strongly-typed attributes
    userID := "user-123"
    requestID := "req-abc-789"
    logger.Info(
        "User logged in",
        slog.String("user_id", userID),
        slog.String("request_id", requestID),
    )
}
```

This would produce output like:
```json
{"time":"2023-10-27T10:00:00Z","level":"INFO","msg":"Hello, structured logs!"}
{"time":"2023-10-27T10:00:00Z","level":"INFO","msg":"User logged in","user_id":"user-123","request_id":"req-abc-789"}
```

This is incredibly powerful. You can now easily find all logs for a specific user or a specific request, even across multiple services.

**Best practice**: Create a logger instance at the start of your application and pass it down to the components that need it, preferably using the `context` package.

## Metrics with Prometheus and Grafana

Metrics are numerical measurements of your system over time. They are aggregated and stored in a time-series database. Metrics are great for understanding the overall health and performance of your system.

-   **Latency**: How long do requests take? (e.g., 99th percentile request latency)
-   **Traffic**: How many requests per second is the service handling?
-   **Errors**: What is the rate of server-side errors (5xx)?
-   **Saturation**: How much of a resource is being used? (e.g., CPU utilization, memory usage)

The de-facto standard for metrics in the cloud-native world is **Prometheus**. It works by having your application expose a `/metrics` endpoint with a list of all its current metrics in a simple text format. The Prometheus server then scrapes this endpoint periodically.

The official Go client library for Prometheus makes this easy:

```go
import (
    "net/http"
    "github.com/prometheus/client_golang/prometheus"
    "github.com/prometheus/client_golang/prometheus/promhttp"
)

// Create a new metric, in this case a counter
var httpRequestsTotal = prometheus.NewCounterVec(
    prometheus.CounterOpts{
        Name: "http_requests_total",
        Help: "Total number of http requests.",
    },
    []string{"method", "path"}, // The labels for this metric
)

func init() {
    // Register the metric with Prometheus's default registry
    prometheus.MustRegister(httpRequestsTotal)
}

func myHandler(w http.ResponseWriter, r *http.Request) {
    // Increment the counter for each request, with the appropriate labels
    httpRequestsTotal.With(prometheus.Labels{"method": r.Method, "path": r.URL.Path}).Inc()
    // ... your handler logic ...
}

func main() {
    // Expose the /metrics endpoint
    http.Handle("/metrics", promhttp.Handler())
    // ... start your server ...
}
```

Once Prometheus is scraping your metrics, you can use a tool like **Grafana** to build dashboards and visualize them. You can create graphs showing your API's latency over time, your error rate, and more.

## Distributed Tracing with OpenTelemetry

Metrics tell you *what* is happening in your system. Logs tell you *why* it's happening at a specific point. **Traces** tell you what happens to a single request as it travels through your entire distributed system.

A trace is composed of a series of **spans**. Each span represents a single unit of work (e.g., an HTTP request, a database call) and has a start time, a duration, and other metadata. When a service makes a call to another service, it propagates a "trace ID" so that the spans from both services can be linked together.

The emerging industry standard for tracing is **OpenTelemetry (OTel)**. OpenTelemetry provides a set of APIs and libraries to instrument your code to generate traces.

Instrumenting your code for tracing can be complex, but the basic idea is:

1.  **Configure an exporter**: This tells the OTel library where to send the traces (e.g., to Jaeger, Zipkin, or a commercial observability platform).
2.  **Start and end spans**: At the beginning of a unit of work (like an HTTP handler), you start a new span from the incoming `context`. You then defer the `span.End()` call.
3.  **Propagate the context**: When you make an outbound call (e.g., another HTTP request or a gRPC call), you must pass the `context` containing the current span information. The OTel instrumentation libraries for `net/http` and `gRPC` can handle this automatically.

```go
// Simplified example of tracing an HTTP handler

func MyTracedHandler(w http.ResponseWriter, r *http.Request) {
    // The OTel HTTP middleware would have started a span for us
    // and put it in the request context.
    ctx := r.Context()

    // Start a new child span for our database call
    _, dbSpan := tracer.Start(ctx, "my-database-call")
    defer dbSpan.End()

    // ... perform database call ...
    dbSpan.SetAttributes(attribute.String("db.statement", "SELECT * FROM users..."))
}
```

When you look at this trace in a tool like Jaeger, you will see a waterfall diagram showing the initial request to your service, followed by the child span for the database call, giving you a clear picture of where time was spent.

## Building Dashboards and Alerts

The final piece of the puzzle is to use this data.

-   **Dashboards**: Use Grafana (or your observability platform's equivalent) to build dashboards that show the key health metrics for your services. You should be able to see latency, traffic, and errors at a glance.
-   **Alerts**: Monitoring is not about staring at dashboards all day. You need to set up alerts to be notified when something is wrong. A good alert is actionable and indicates a real problem (e.g., "99th percentile latency for the user-service has been above 200ms for 5 minutes" or "The error rate for the payment-service is above 1%").

Observability is a deep topic, but by implementing these three pillars, you gain the ability to understand and debug even the most complex distributed systems, which is a non-negotiable skill for a senior engineer.
