---
title: Observability
---

## Observability in Microservices Architecture

### What Is Observability?

**Observability** in microservices refers to engineering systems that emit
enough telemetry—metrics, logs, and traces—to allow operators and developers to
understand the system’s internal states through its external outputs. This is
distinct from mere monitoring: while monitoring typically answers, “Is the
system up?”, observability asks deeper questions, such as “Why is it
misbehaving?” and “Where is the failure or performance bottleneck?”.

### Dimensions of Observability

The “**Three Pillars of Observability**” are:

- **Metrics**: Numeric measures that represent a system’s health/performance
  over time (e.g., latency, throughput, error rates).
- **Logs**: Immutable, timestamped records of discrete events or errors.
- **Traces**: Chain of events that follow a user request or transaction across
  multiple services.

In practice, successful observability strategies also integrate **events** and
**metadata/correlation IDs** to enhance cross-service diagnostics.

### Importance of Observability in Microservices

Highly distributed architectures mean failure points are more numerous and
complex to trace. **Observability is essential for:**

- **Rapid troubleshooting:** Diagnosing the root cause of outages or high
  latencies
- **Performance tuning:** Pinpointing bottlenecks at service or infrastructure
  levels
- **Incident response:** Coordinating response and recovery using up-to-date
  telemetry
- **Ensuring reliability:** Validating resilience and robustness of services
  before deployment to production environments
- **Enabling automation and self-healing:** Automated systems rely on observable
  signals for scaling, failover, or rollback decisions
- **Continuous improvement:** Enabling feedback loops for architectural
  iteration

Without investing in observability, organizations risk long downtimes, poor user
experience, and slow incident resolution, as indicated in many postmortems of
large-scale outages.

### Observability in ASP.NET Core Microservices

**ASP.NET Core** offers a modern, cross-platform framework well-suited for
microservices. It natively supports:

- **Health checks** via the `Microsoft.AspNetCore.Diagnostics.HealthChecks`
  package
- **Metrics, Logs, and Traces** with integrations for OpenTelemetry, Prometheus,
  Jaeger, Zipkin, Application Insights, and more
- Reverse proxies and API gateways, providing hooks for service mesh integration

---

## Metrics in Microservices

### Definition and Examples

**Metrics** are quantitative measures describing various aspects of a system’s
health or performance. They can be:

- **Infrastructure metrics**: CPU, memory, disk, networking
- **Application metrics**: Request duration, error rate, success/failure counts,
  queue depth
- **Business metrics**: Transactions processed, dollar value, conversion rates

Metrics are typically **numeric, time-series data points** collected and
analyzed for trends, thresholds, and anomalies. In microservices, they are
usually tagged (with labels such as service name, version, environment) and
published to systems like Prometheus, which then enable rich querying and
alerting.

### Importance of Metrics

Key uses include:

- **Monitoring and alerting:** Promptly notifying operators of unhealthy states
- **Capacity planning:** Understanding baseline and peak loads
- **Performance tuning:** Locating high-latency endpoints or error hotspots
- **SLI/SLO tracking:** Supporting Site Reliability Engineering practices for
  reliability and uptime commitments

In microservices, metrics help identify issues both vertically (in a single
service) and horizontally (across dependencies).

### Essential Microservices Metrics

It’s crucial to focus on **actionable** metrics. For ASP.NET Core microservices,
best practices include tracking:

- **Request rates (RPS/QPS)**
- **Request latency (response times)**
- **Error rate (4xx/5xx per endpoint/service)**
- **Dependency/service call latencies**
- **Resource utilization (CPU%, RAM, container stats)**
- **Custom business metrics (orders, signups, payments processed)**

These metrics should be **granular** (by service, endpoint, and label),
**accurate**, and **timely** for effective operational response.

### Metrics in ASP.NET Core

ASP.NET Core supports instrumentation through:

- **OpenTelemetry SDK**
- **prometheus-net** middleware
- **.NET built-in event counters**
- **Custom middleware to expose application-specific metrics**

Sample code to collect HTTP request duration:

```csharp
using Prometheus;

public class Startup
{
    private static readonly Histogram RequestDuration = Metrics.CreateHistogram(
        "http_request_duration_seconds",
        "Duration of HTTP requests in seconds.",
        new HistogramConfiguration
        {
            LabelNames = new[] { "method", "path", "status_code" }
        });

    public void Configure(IApplicationBuilder app)
    {
        app.Use(async (context, next) =>
        {
            var path = context.Request.Path;
            var method = context.Request.Method;
            var sw = Stopwatch.StartNew();
            await next();
            sw.Stop();
            RequestDuration
                .WithLabels(method, path, context.Response.StatusCode.ToString())
                .Observe(sw.Elapsed.TotalSeconds);
        });

        app.UseMetricServer(); // Exposes /metrics endpoint
    }
}
```

This setup provides Prometheus-compatible metrics for scraping and
visualization.

---

## Health Checks in ASP.NET Core

### What Are Health Checks?

**Health checks** are lightweight endpoints or routines that verify if services
and their dependencies (databases, caches, message brokers) are running as
expected. They answer, “Is this service healthy or degraded?” and can be used
both for **liveness** (is the app running?) and **readiness** (can the app serve
traffic?) probes .

### Why Are Health Checks Essential?

In microservices:

- **Orchestration:** Health checks enable orchestrators (Kubernetes, Docker
  Swarm) to restart unhealthy containers and route traffic only to ready
  instances.
- **Layered health:** They can check not just the service itself but critical
  dependencies, detecting partial outages.
- **Autoscaling:** Readiness probes prevent introducing unhealthy replicas
  during scaling events.

Without robust health checks, applications risk failing silently or overwhelming
backends with failing requests.

### Health Check Types

- **Liveness checks:** Is the application process running? Detects unresponsive
  or deadlocked apps.
- **Readiness checks:** Is the service _and its dependencies_ ready to receive
  traffic?
- **Custom checks:** Business logic, upstream API connectivity, or queue states.

### Health Checks in ASP.NET Core: Implementation

ASP.NET Core (since v2.2) offers a robust Health Checks API via the
`Microsoft.Extensions.Diagnostics.HealthChecks` namespace. All health checks are
typically exposed via an `/health` endpoint.

#### Registering Health Checks

```csharp
// Startup.cs
public void ConfigureServices(IServiceCollection services)
{
    services.AddHealthChecks()
            .AddSqlServer(Configuration["ConnectionStrings:DefaultConnection"]) // DB check
            .AddRedis(Configuration["Redis:ConnectionString"]) // Cache check
            .AddCheck<CustomHealthCheck>("custom"); // Custom checks
}
```

#### Mapping the Health Endpoint

```csharp
public void Configure(IApplicationBuilder app)
{
    app.UseRouting();
    app.UseEndpoints(endpoints =>
    {
        endpoints.MapHealthChecks("/health");
    });
}
```

This exposes `/health`, providing a JSON or plain text status response.

#### Example JSON Output

```json
{
  "status": "Healthy",
  "results": {
    "sqlserver": { "status": "Healthy" },
    "redis": { "status": "Healthy" },
    "custom": { "status": "Healthy" }
  }
}
```

#### Custom Health Check

```csharp
using Microsoft.Extensions.Diagnostics.HealthChecks;

public class CustomHealthCheck : IHealthCheck
{
    public Task<HealthCheckResult> CheckHealthAsync(
        HealthCheckContext context,
        CancellationToken cancellationToken = default)
    {
        // Custom logic here
        bool isHealthy = true;
        if (isHealthy)
            return Task.FromResult(HealthCheckResult.Healthy("The check indicates a healthy result."));
        return Task.FromResult(HealthCheckResult.Unhealthy("The check indicates an unhealthy result."));
    }
}
```

### Best Practices for Health Checks

- **Separation of Liveness vs. Readiness:** Avoid using the same logic for both.
  Killing an app for a temporary backend blip can worsen outages.
- **Lightweight and Fast:** Health checks should return quickly to avoid
  timeouts or cascading failures.
- **Dependency Awareness:** Include checks for critical external dependencies.
- **Non-Blocking:** Health check endpoints should never block the main
  application threads.
- **Security:** Use authentication/authorization, especially in production, to
  prevent leaking internal status to attackers.

---

## How Observability Improves Reliability and Performance

### System Reliability

**Reliability** is the assurance that a system will perform its required
functions under stated conditions for a specified period. Observability directly
supports reliability by:

- **Detecting failures:** Rapid identification and isolation of faults
- **Validating SLAs/SLOs/SLIs:** Data-driven uptime/performance guarantees
- **Enabling proactive remediation:** Automated healing and fallback routines
  triggered by observable signals
- **Supporting incident response:** Speeding up Mean Time To Detect (MTTD) and
  Mean Time To Recovery (MTTR)
- **Survivability in the face of partial failures:** Observability enables
  patterns such as circuit breakers and fallback logic

### System Performance

Performance encompasses responsiveness (latency), throughput, and resource
efficiency. Observability impacts performance by:

- **Continuous monitoring of key metrics:** Quick detection of performance
  regressions or abnormal latency
- **Bottleneck identification:** Pinpointing slowest services or endpoints in
  distributed traces
- **Capacity planning:** Ensuring horizontal scaling matches actual demand,
  reducing both overprovisioning and resource starvation
- **A/B and canary testing:** Validating new releases in production with live
  metrics/traces

In summary, observability is **foundational** to achieving high reliability and
performance in microservices, enabling confident operations and rapid iteration.

---

## Core Observability and Monitoring Concepts

### Distributed Tracing

**Definition:** Distributed tracing tracks a single request as it traverses
multiple microservices, illuminating the “path” of the request, including timing
and status at each hop. This is vital for understanding complex workflows and
pinpointing latency or errors across service boundaries.

- **Trace:** The complete journey of a request.
- **Span:** A single logical operation (e.g., database call) within a trace,
  uniquely identified and correlated.

**Example technologies:** OpenTelemetry, Jaeger, Zipkin, AWS X-Ray.

**ASP.NET Core Implementation (OpenTelemetry):**

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddOpenTelemetryTracing(builder => builder
        .AddAspNetCoreInstrumentation()
        .AddHttpClientInstrumentation()
        .AddSqlClientInstrumentation()
        .SetSampler(new AlwaysOnSampler())
        .AddJaegerExporter() // Or Zipkin, OTLP, etc.
    );
}
```

**Benefit:** Drastically improves the ability to debug, optimize, and ensure
correctness in service-to-service interactions.

### Telemetry

**Definition:** Telemetry refers to the automated collection, transmission, and
processing of instrumentation signals (metrics, logs, spans) from applications
and infrastructure. It underpins observability by making data available for
external analysis in real time.

**OpenTelemetry** is the current standard for telemetry in cloud-native
systems—it unifies logs, metrics, and traces instrumentation across language
runtimes, including .NET.

### Service Mesh

**Definition:** A service mesh is a dedicated infrastructure layer that handles
service-to-service communication, observability, and security in a transparent
way, typically via network proxies (such as Istio, Linkerd). It enables:

- **Traffic management:** Routing, load balancing, canary deployments, failover
- **Policy enforcement:** mTLS, RBAC, quotas, network policies
- **Automatic telemetry:** Out-of-the-box metrics, distributed traces, and logs
  for all services without code changes

**Observability Benefits:** Service mesh provides uniform, zero-code
observability and traffic metrics, facilitating global views and security
enforcement.

**Typical Interview Scenario:** “How would you implement cross-service tracing
without modifying application code?” (Answer: Deploy a service mesh with native
tracing support.)

### Fault Tolerance

**Definition:** Fault tolerance is the ability of a system to gracefully recover
from failures in components or dependencies. Patterns include:

- **Retry:** Transparent retries for transient errors.
- **Circuit breaker:** Stops making requests to failing services, providing
  fallback or error response.
- **Bulkhead:** Isolates service resources to limit failure scope.
- **Timeouts:** Ensures that client calls don’t block indefinitely on downstream
  services.

**Observability Tie-In:** Telemetry from metrics and health checks often
trigger, tune, or verify the efficacy of fault tolerance strategies.

---

### Key Topics and Definitions for Observability & Monitoring

| Term                | Definition and Relevance in Microservices               | Example/Importance                                   |
| ------------------- | ------------------------------------------------------- | ---------------------------------------------------- |
| Metrics             | Numeric data about system/app performance               | Latency, error rates, throughput                     |
| Logs                | Timestamped, text-based records of events/errors        | Error logs, audit trails                             |
| Traces              | End-to-end request flows across multiple services       | Diagnose slow paths, cross-service correlation       |
| Distributed Tracing | Tracking a request across services with unique IDs      | Pinpointing bottlenecks, debugging transaction flows |
| Telemetry           | Automated collection of metrics, logs, and traces       | Foundation for observability pipelines               |
| Health Checks       | Liveness/readiness indicators, usually via endpoint     | /health endpoints, Kubernetes readiness probes       |
| Service Mesh        | Layer for service comms (proxy), telemetry, and auth    | Istio, Linkerd: zero-code observability and security |
| Fault Tolerance     | System resilience patterns (retry, circuit breaker)     | Ensures graceful degradation, automatic recovery     |
| Correlation ID      | Unique identifier passed across services/logs/traces    | Log enrichment, joins logs/traces for a request      |
| Prometheus          | Open-source monitoring & alerting metric system         | Metric scraping, time-series DB                      |
| Grafana             | Visualization and dashboarding of observability data    | Real-time dashboards and alerting                    |
| OpenTelemetry       | Open standard for traces, metrics, logs instrumentation | Unified .NET observability                           |

---

## Monitoring Tools and Ecosystem: Comparison and Best Practices

### Monitoring and Observability Tools Landscape

Modern microservices observability relies on a robust and extensible tooling
ecosystem. Below is a comparative summary of widely adopted monitoring
solutions.

| Tool/Framework       | Primary Role                                   | Strengths                           | Integration with ASP.NET Core        |
| -------------------- | ---------------------------------------------- | ----------------------------------- | ------------------------------------ |
| OpenTelemetry        | Unified tracing, metrics, logs instrumentation | Open standard, vendor-agnostic      | SDK, .NET Instrumentation libraries  |
| Prometheus           | Metrics collection and alerting                | Rich query (PromQL), time-series DB | prometheus-net, OpenTelemetry export |
| Grafana              | Visualization, alerting dashboard              | Extensible plugins, real-time views | Data sources: Prometheus, OTEL       |
| Jaeger/Zipkin        | Distributed tracing backend                    | Visualization, trace search         | Jaeger .NET exporter, OpenTelemetry  |
| Application Insights | App monitoring, trace, log collection          | Deep Azure integration              | SDK, seamless with .NET/ASP.NET Core |
| Fluentd/Logstash     | Log aggregation and routing                    | Data pipeline extensibility         | Log enrichment, forwarding           |

### Best Practices for Observability Tooling

- **Leverage Open Standards**: Use OpenTelemetry and Prometheus to avoid vendor
  lock-in.
- **Automate Instrumentation**: Integrate at SDK/middleware level to reduce code
  complexity.
- **Tag All Telemetry with Correlation IDs**: This enables powerful,
  context-aware diagnostics.
- **Centralize Dashboards**: Aggregate views across all services for unified
  insights.
- **Integrate Alerting**: Use Prometheus and Grafana rules for automated
  notifications.
- **Secure Observability Endpoints**: Prevent attackers from accessing sensitive
  operational data.

---

## Step-by-Step Guide: Implementing Observability in ASP.NET Core Microservices

Combining the concepts above, this section provides an actionable, code-centric
guide for setting up **health checks**, **OpenTelemetry tracing/metrics**,
**Prometheus metric scraping**, and **Grafana dashboards**, focused on ASP.NET
Core applications.

### 1. Setting Up Health Checks in ASP.NET Core

**A. Install NuGet Packages**

- `Microsoft.Extensions.Diagnostics.HealthChecks`
- `AspNetCore.HealthChecks.UI` (for dashboard, optional)
- Dependency-specific health checks: e.g., `AspNetCore.HealthChecks.SqlServer`

```bash
dotnet add package Microsoft.Extensions.Diagnostics.HealthChecks
dotnet add package AspNetCore.HealthChecks.SqlServer
```

**B. Register Health Checks in Program/Startup**

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddHealthChecks()
        .AddSqlServer(Configuration["Db:ConnectionString"], name: "sqlserver")
        .AddRedis(Configuration["Redis:ConnectionString"], name: "redis")
        .AddCheck<CustomHealthCheck>("custom");
}
```

**C. Map Health Endpoint**

```csharp
public void Configure(IApplicationBuilder app)
{
    app.UseRouting();

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapHealthChecks("/healthz", new HealthCheckOptions
        {
            // Customize output or status codes if desired
        });
    });
}
```

**D. Customize Health Check Output (Optional)**

```csharp
options.ResponseWriter = async (context, report) =>
{
    context.Response.ContentType = "application/json";
    var result = JsonConvert.SerializeObject(
        new {
            status = report.Status.ToString(),
            details = report.Entries.Select(e => new {
                key = e.Key,
                value = e.Value.Status.ToString(),
                description = e.Value.Description
            })
        });
    await context.Response.WriteAsync(result);
};
```

**E. Kubernetes/Container Consideration**

Use `/healthz` for **liveness** and `/ready` for **readiness** by adding
endpoint filters or configuring different sets of checks per endpoint.

**F. Health Checks UI**

Optionally add `AspNetCore.HealthChecks.UI` for a dashboard visualization and
centralized status panel.

### 2. Enabling OpenTelemetry (Tracing & Metrics) in ASP.NET Core

**A. Install NuGet Packages**

```bash
dotnet add package OpenTelemetry
dotnet add package OpenTelemetry.Extensions.Hosting
dotnet add package OpenTelemetry.Instrumentation.AspNetCore
dotnet add package OpenTelemetry.Exporter.Prometheus
dotnet add package OpenTelemetry.Exporter.Jaeger
```

**B. Configure OpenTelemetry Tracing**

Add to `Program.cs` or `Startup.cs`:

```csharp
builder.Services.AddOpenTelemetry()
    .WithTracing(tracing =>
    {
        tracing
            .AddAspNetCoreInstrumentation()
            .AddHttpClientInstrumentation()
            .AddSource("MyApp.CustomSource")
            .SetSampler(new AlwaysOnSampler())
            .AddJaegerExporter(jaegerOpts =>
            {
                jaegerOpts.AgentHost = "localhost";
                jaegerOpts.AgentPort = 6831;
            });
    });
```

**C. Configure OpenTelemetry Metrics**

```csharp
builder.Services.AddOpenTelemetry()
    .WithMetrics(metrics =>
    {
        metrics
            .AddAspNetCoreInstrumentation()
            .AddHttpClientInstrumentation()
            .AddPrometheusExporter(); // Exposes /metrics endpoint
    });
```

**D. Expose Prometheus Metrics Endpoint**

```csharp
app.UseOpenTelemetryPrometheusScrapingEndpoint();
// This usually defaults to `/metrics`
```

**E. Optional: Log Enrichment**

For logs, correlate with trace IDs (see Serilog, NLog, or
Microsoft.Extensions.Logging with custom middleware):

```csharp
public class CorrelationIdMiddleware
{
    public async Task InvokeAsync(HttpContext context, RequestDelegate next)
    {
        var correlationId = Guid.NewGuid().ToString();
        context.Items["CorrelationId"] = correlationId;
        context.Response.Headers.Add("X-Correlation-ID", correlationId);
        await next(context);
    }
}
```

Enrich structured logs with trace IDs for end-to-end diagnostics.

### 3. Integrating Prometheus for Metrics Collection

**A. Deploy Prometheus (Docker or Binary)**

**prometheus.yml example:**

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: "aspnetcore-microservice"
    static_configs:
      - targets: ["myservice:5000"] # Update with service address
```

**B. Run Prometheus**

```bash
docker run -p 9090:9090 -v ./prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus
```

Prometheus will start scraping `/metrics` endpoint of all mapped targets.

### 4. Grafana: Visualizing Metrics and Building Dashboards

**A. Deploy Grafana (Docker recommended)**

```bash
docker run -d -p 3000:3000 grafana/grafana
```

**B. Configure Data Source**

- Add Prometheus as a data source in Grafana dashboard settings.
- Set URL to `http://localhost:9090` (or address as per deployment).

**C. Import ASP.NET Core Sample Dashboard**

Grafana’s community dashboard 10915:
https://grafana.com/grafana/dashboards/10915-asp-net-core-controller-summary-prometheus/

- Download/import dashboard JSON, point queries to your Prometheus data source.
- Visualize HTTP request rates, latencies, error counts by controller/action.

**D. Best Practices for Dashboarding**

- Focus on high-signal metrics (rates, errors, latency p50/p95/p99).
- Use templating/variables for service, environment, or version breakdown.
- Add alerts for thresholds: e.g., error rate > 5%, p95 latency > 1s.
- Keep dashboards concise but comprehensive; link logs and traces where
  possible.

---

## Sample Full Implementation: Observability in ASP.NET Core

Below is a succinct, fully integrated sample code (Program.cs, .NET 6/7 style):

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();
builder.Services.AddHealthChecks()
    .AddSqlServer(builder.Configuration["ConnectionStrings:MainDb"], name: "sqlserver");

builder.Services.AddOpenTelemetry()
    .WithTracing(tracing =>
    {
        tracing
            .AddAspNetCoreInstrumentation()
            .AddJaegerExporter();
    })
    .WithMetrics(metrics =>
    {
        metrics
            .AddAspNetCoreInstrumentation()
            .AddPrometheusExporter();
    });

var app = builder.Build();

app.UseOpenTelemetryPrometheusScrapingEndpoint();
app.MapHealthChecks("/healthz");
app.MapControllers();
app.Run();
```

**Prometheus config:**

```yaml
scrape_configs:
  - job_name: "aspnetcore"
    static_configs:
      - targets: ["host.docker.internal:5000"]
```

**Grafana steps:**

- Connect Prometheus, import dashboard 10915, and analyze live data.

---

## Advanced Topics: Alerting, Logging, Configuration, and Performance Impact

### Alerting

- **Prometheus AlertManager** enables threshold-based alerts (e.g., “if 5xx
  error rate > 2% in 5m window, page devops”).
- **Grafana** supports rule-based alerts, notifying teams via email, Slack,
  PagerDuty, etc.
- Always **test alert sensitivity**—ensure you get actionable, not noisy, pages.

### Logging Integration for Observability

- Use structured logging (Serilog, NLog) rather than plain text.
- Correlate log statements with **trace/context IDs** for distributed tracing.
- Use log sinks (Elasticsearch, Loki, Azure Monitor) for search and diagnosis.
- Enrich logs at middleware level, so all events carry request/trace metadata.

### Configuration Management for Observability

- Use **environment-based configuration** for observability endpoints and
  exporters.
- Securely store sensitive telemetry endpoints (OTLP, Jaeger, Prometheus) via
  secrets/config maps.
- Enable or suppress specific instrumentation via config for performance tuning.

### Visualization Best Practices

- Tailor dashboards for roles (ops, SRE, dev, exec).
- Implement **drill-down panels**: high-level service health links to
  endpoint-level details.
- Document dashboards and metric definitions.
- Regularly review dashboard relevance—retire or update as the system evolves.

### Performance Overhead and Optimization

- **Minimal impact when done right:** Modern telemetry exporters stream metrics
  asynchronously, minimizing latency impact.
- Instrumentation should be **selective**: instrument critical paths, not every
  low-level routine.
- **Sampling**: Trace only a subset of requests in high-volume production to
  contain performance costs.
- Monitor **resource utilization** (CPU, RAM) to ensure observability tooling
  doesn’t become a bottleneck.
- Review exporter/collector configuration as system load grows.

For further context, empirical studies (such as recent research on tracing
overhead) show OpenTelemetry and Prometheus, if sampled and batched properly,
add acceptably low overhead (< 2-3%) on request latency in most real-world
scenarios, especially compared to the operational benefit they provide.
