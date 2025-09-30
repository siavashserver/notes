---
title: Http Client Factory
---

Short answer up front: **don’t make lots of short-lived `HttpClient` instances**
— reuse them or use `IHttpClientFactory`. `HttpClient` itself is a lightweight
wrapper, but its _underlying handlers/sockets_ are heavy. Creating/disposing
many clients creates many connection pools and TCP sockets which can quickly
exhaust OS resources (ephemeral ports / file descriptors / TIME_WAIT entries).
`IHttpClientFactory` gives you safe reuse, pooling, and configurable lifetimes
so you avoid socket exhaustion while still handling DNS changes and other
issues.

# Why this happens (what’s going on under the hood)

- `HttpClient` uses an **HttpMessageHandler** (on .NET Core the default is
  **SocketsHttpHandler**) that manages connections and performs the TCP work.
  The handler owns the socket connection pool (keep-alive, reuse, timeouts).
- If you create a new `HttpClient` (and thus a new handler) for every request,
  you create many separate connection pools. Each pool may open TCP connections
  that sit in various TCP states (including `TIME_WAIT`) after they close.
- TCP resources are finite:

  - **Ephemeral source ports** (the client-side port used for outbound
    connections) are limited by OS configuration — when each new outbound TCP
    needs a new source port, you can run out.
  - **File descriptor / socket limits** (process or system limits) can be hit.
  - **TIME_WAIT**: sockets that are closed can remain in `TIME_WAIT` for a
    while, preventing immediate reuse of that (ip,port) tuple and consuming
    ephemeral ports.

- Symptoms of exhaustion: `SocketException: Address already in use`, many
  requests blocked/very slow, `Too many open files`, high counts of `TIME_WAIT`
  in `netstat`.

# Why you can’t just `new HttpClient()` per request even though it’s `IDisposable`

- Disposing `HttpClient` will eventually free resources, but the **TCP
  connections** managed by the handler may still be in TIME_WAIT or the handler
  may maintain the pool. On many runtimes disposing a client frequently leads to
  short-lived handlers and many sockets being created and torn down — the OS
  will still hold sockets in TIME_WAIT and you’ll exhaust ephemeral ports or
  file descriptors long before the OS clears them.

# What `IHttpClientFactory` (HttpClientFactory) solves

- **Centralized pooling** of `HttpMessageHandler` instances so
  sockets/connection pools are reused rather than recreated for every
  `HttpClient`.
- **Configurable handler lifetime** (rotate handlers periodically) so you don’t
  keep a single handler forever — this avoids stale DNS entries while still
  reusing connections most of the time.
- Built-in patterns: named clients, typed clients, typed configuration.
- Integrates easily with logging and resilience libraries (Polly) and lets you
  configure settings like `MaxConnectionsPerServer` or pooled connection
  lifetime in one place.
- Reduces memory/sockets leaks and the risk of socket exhaustion while keeping
  correct behavior for DNS updates, TLS, etc.

# Practical knobs you can use

- Reuse a single `HttpClient` instance for many requests (singleton) in
  non-ASP.NET apps.
- Prefer `IHttpClientFactory` in ASP.NET Core (register with DI):

  ```csharp
  services.AddHttpClient("github", c => {
      c.BaseAddress = new Uri("https://api.github.com/");
      c.DefaultRequestHeaders.Add("User-Agent", "MyApp");
  })
  .ConfigurePrimaryHttpMessageHandler(() => new SocketsHttpHandler {
      MaxConnectionsPerServer = 10,
      PooledConnectionLifetime = TimeSpan.FromMinutes(5) // rotate handler periodically
  });
  ```

  Then inject `IHttpClientFactory` or `HttpClient` (typed client) where needed.

- Tune `MaxConnectionsPerServer` when you need parallelism to a single host.
- Tune `PooledConnectionLifetime` (or equivalent) so handlers are recycled
  occasionally to pick up DNS changes.

# “What’s the maximum cap?” — there’s no single .NET limit

There isn’t a single magic cap enforced by .NET; limits are imposed by the OS
and by handler settings:

- **Ephemeral port range (OS-level)** — limits how many _concurrent outbound_
  connections can use distinct source ports. Typical defaults (varies by OS and
  version):

  - Windows often uses an ephemeral range like 49152–65535 (~16k ports) by
    default.
  - Linux distributions often use a range like 32768–60999 (~28k) by default.
    These ranges are configurable on the OS; changing them increases or
    decreases the available ephemeral ports.

- **File descriptor / handle limits** — each socket uses a file descriptor. On
  Linux `ulimit -n` or systemd limits control per-process fd limits. On Windows
  there are handle limits and other resources that become relevant.
- **Per-host connection limit** — in old .NET Framework the
  `ServicePointManager.DefaultConnectionLimit` default was low (2) for HTTP/1.1;
  on .NET Core the `SocketsHttpHandler.MaxConnectionsPerServer` default allows
  much more concurrency (practically an unbounded default), but you can (and
  should) configure it to something reasonable for your app and target servers.
- **TIME_WAIT lifetime** — TCP TIME_WAIT is typically 60–120 seconds (OS
  dependent). Connections in TIME_WAIT consume ephemeral ports until they
  expire.

So your “maximum” concurrent outbound TCP connections is essentially:
`min(ephemeral_ports_available, file_descriptors_available, any configured
MaxConnectionsPerServer × number_of_distinct_remote_endpoints)`.

# How to debug/observe socket exhaustion

- `netstat -anp | grep TIME_WAIT` (Linux) or `netstat -ano` (Windows) to see
  many `TIME_WAIT` sockets or many sockets to the same remote host.
- OS metrics: number of open file descriptors, ephemeral port usage,
  TcpTimedWaitDelay on Windows, etc.
- Application logs of `SocketException`.

# Concrete recommended rules

- In ASP.NET Core: **use `IHttpClientFactory`** (register via `AddHttpClient`) —
  it’s the recommended pattern.
- In console/desktop apps: reuse a single `HttpClient` instance per logical
  configuration (or implement your own factory that reuses handlers).
- If you need high throughput to a single server, set `MaxConnectionsPerServer`
  to a sensible value rather than leaving it effectively unlimited or too low.
- If you must create many handlers (rare), configure `PooledConnectionLifetime`
  (rotate after N seconds) to avoid stale DNS while keeping pooling.
- Prefer HTTP/2 where possible — it can multiplex many requests over a single
  TCP connection, drastically reducing socket usage.
