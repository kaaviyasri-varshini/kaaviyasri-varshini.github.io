Title: Logs, Metrics, Traces and Spans: The Building Blocks of Observability
Date: 2026-09-20
Category: Observability
Tags: logs, metrics, traces, spans, observability, monitoring, debugging
Slug: logs-metrics-traces-spans
Featured_Image: /images/logs-metrics-traces-spans.png

When something breaks in production, you need to answer three questions: what happened, how bad is it, and where did it go wrong? Logs, metrics, and traces each answer one of these. Spans are the pieces that make traces work. Together they are the core of observability.

## Logs

**Log** — A timestamped record of a single event that happened in your system, such as a request received, an error thrown, or a job finished.
Logs are the most detailed signal you have. They tell you exactly what happened at a specific moment, which makes them the first place to look when debugging.

**Structured Logging** — Writing logs as key-value data (usually JSON) instead of free-form text.
Fields like `user_id`, `request_id`, and `level` make logs searchable and filterable. Plain text logs are hard to query at scale.

**Log Levels** — Labels such as DEBUG, INFO, WARN, and ERROR that describe how important an entry is.
Levels let you keep noise down in production while still capturing the errors that matter.

## Metrics

**Metric** — A numeric measurement collected over time, like request count, CPU usage, or error rate.
Metrics are cheap to store and fast to query because they are aggregated numbers, not individual events. They are best for dashboards, trends, and alerts.

**Counter, Gauge, Histogram** — The three common metric types.
A counter only goes up (total requests), a gauge moves up and down (memory in use), and a histogram records the distribution of values (request latency buckets). Histograms are how you get percentiles like p95 and p99.

**Cardinality** — The number of unique label combinations a metric has.
Adding a label like `user_id` can multiply your time series into millions. High cardinality is the most common reason metrics systems become slow and expensive.

## Traces

**Trace** — The complete journey of one request as it moves through all the services it touches.
A single user action may hit an API gateway, an auth service, a database, and a cache. A trace connects them all so you can see the whole path in one view.

**Trace ID** — A unique identifier shared by every step in the same request.
It is passed between services, usually through HTTP headers, so each service can attach its work to the same trace. This is called context propagation.

**Distributed Tracing** — Tracing that works across service and machine boundaries.
It is the only practical way to find which service is slow in a microservices system, since no single service's logs show the whole picture.

## Spans

**Span** — One unit of work inside a trace, such as an HTTP call, a database query, or a function execution.
Each span records a name, a start time, a duration, and a status. A trace is simply a collection of spans.

**Parent and Child Spans** — Spans nest to show what called what.
A request span might have child spans for an auth check and a DB query. This tree structure produces the waterfall view that shows exactly where time was spent.

**Span Attributes and Events** — Extra data attached to a span.
Attributes are key-value pairs like `http.status_code=500` or `db.table=orders`. Events are timestamped notes inside the span, such as an exception being raised or a retry starting.

## How They Fit Together

**Metrics Tell You Something Is Wrong** — An alert fires because error rate or latency crossed a threshold.
This is your starting signal. It shows the size of the problem but not the cause.

**Traces Tell You Where** — You open a slow or failed trace and find the span that took too long or errored.
This narrows the search from the whole system down to one service or one call.

**Logs Tell You Why** — You read the logs for that service, ideally filtered by the trace ID, to find the exact error and its context.
Linking logs to trace IDs is the single most useful thing you can do to connect the three signals.

## Wrapping Up

Logs give detail, metrics give scale, traces give the path, and spans are the steps along it. None of them replaces the others. Start with structured logs, add a few key metrics, then introduce tracing once you have more than one service. OpenTelemetry is a good vendor-neutral way to collect all of them with one standard.