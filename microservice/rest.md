---
title: REST
---

REST (Representational State Transfer) is an architectural style for networked
applications.

## RESTful Service Constraints

A RESTful service is a web service that strictly follows the REST architectural
style.

- Client–Server: Separates user interface (client) from data storage (server).
  Each can evolve independently, boosting scalability and maintainability.
- Statelessness: Each request must include all information needed; servers don’t
  track client state. This makes scaling simpler and requests parallelizable.
- Cacheable: Responses must specify caching behavior (Cache-Control, ETag),
  improving performance and reducing server load.
- Layered System: The architecture can include layers like load balancers,
  caches, security gateways, etc.
- Code-on-Demand **(optional)**: Servers may provide executable code (e.g.,
  JavaScript) to extend client functionality. This is optional but allowed.
- Uniform Interface: Simplifies system architecture via a consistent,
  resource-based interface using URIs and standard HTTP verbs.
  - Resource identification via URIs
  - Manipulation through representations (JSON, XML, HTML)
  - Self‑descriptive messages (media types, headers)
  - HATEOAS (Hypermedia as the Engine of Application State): Enables clients to
    dynamically interact with an API using links and media types in responses,
    rather than relying on hard-coded URIs or out-of-band information.
    - Entry Point: Clients begin with a fixed URL: `/accounts/12345`
    - Discoverable Actions via Hypermedia: Server responses include embedded
      links (`rel` + `href`) that indicate available actions:
      ```json
      {
        "account": { "account_number": 12345, "balance": 100.0 },
        "links": {
          "deposits": "/accounts/12345/deposits",
          "withdrawals": "/accounts/12345/withdrawals",
          "transfers": "/accounts/12345/transfers",
          "close-requests": "/accounts/12345/close-requests"
        }
      }
      ```
    - State-Driven Navigation: As the application state changes, the returned
      links adapt correspondingly. For instance, an overdrawn account might only
      offer a `deposit` link next.

## HTTP Verbs

- GET: Retrieve a resource. Safe, idempotent, cacheable.
- HEAD: Same as GET but no response body. Safe, idempotent, cacheable.
- POST: Submit data to a resource, potentially causing a change. Not safe, not
  idempotent.
- PUT: Replace the entire resource. Not safe, but idempotent.
- PATCH: Partially update a resource. Not safe, non-idempotent.
- DELETE: Remove a resource. Not safe, but idempotent.
- OPTIONS: Describe the communication options for the target resource. Safe,
  idempotent.
- TRACE: Echo back the received request (mainly for diagnostics). Safe,
  idempotent.
- CONNECT: Establish a tunnel (e.g., for HTTPS). Not safe, not idempotent.

In HTTP, the term **Safe** refers to methods that are _intended to be
read-only_, meaning they _should not modify_ the server's state or resources.
While **all safe methods are idempotent**, **not all idempotent methods are
safe**.

_`POST`, `PUT`, `PATCH`, `DELETE` and `CONNECT` are not safe._

_`POST`, `PATCH` and `CONNECT` are not idempotent._

## Idempotency

An HTTP method is considered idempotent if making multiple identical requests
has the same effect on the server's state as making a single request,regardless
of how many times it's called.

Idempotency guarantees **safe retrying** and **predictable server state**,
critical for distributed systems and error handling.

## Cacheability Headers

Cacheability allows storing copies of responses at clients or intermediary
caches (e.g., browsers, CDNs, proxies), reducing server load, bandwidth usage,
and latency.

### Cache-Control

- `max-age=seconds:` validity period.
- `s-maxage=seconds`: TTL for shared caches (overrides max-age).
- `public`: cacheable by any cache.
- `private`: cacheable only by client (browser).
- `no-cache`: must revalidate with origin before reuse.
- `no-store`: must not be cached anywhere.
- `must-revalidate` / `proxy-revalidate`: stale entries must be revalidated.
- `immutable`: response won’t change; safe to skip validation.
- `stale-while-revalidate`, `stale-if-error`: serve stale while revalidating or
  on server error.

```
Cache-Control: public, max-age=3600, stale-while-revalidate=60
```

### ETag

A version identifier for a resource (typically a hash). Enables _conditional
requests_:

- Server returns `ETag: "xyz"`
- Client re-sends with `If-None-Match: "xyz"`
  - If unchanged -> `304 Not Modified` (response body omitted)
  - If changed -> new content with new ETag

### Last-Modified / If-Modified-Since

Another validator based on timestamps. Works similarly: server checks if content
is newer than the given date before sending content or a `304`

### Expires

Legacy header indicating an absolute expiry date. Superseded by `max-age`, but
still used for older caches.

## HTTP Status Codes

### 1xx Informational

100 Continue, 101 Switching Protocols, 102 Processing, 103 Early Hints

### 2xx Success

200 OK, 201 Created, 202 Accepted (Request accepted, processing not yet done),
203 Non‑Authoritative Information, 204 No Content, 205 Reset Content, 206
Partial Content, 207 Multi‑Status, 208 Already Reported, 226 IM Used (server is
returning a delta in response to a GET request)

### 3xx Redirection

300 Multiple Choices, 301 Moved Permanently, 302 Found, 303 See Other, 304 Not
Modified, 307 Temporary Redirect, 308 Permanent Redirect

### 4xx Client Errors

400 Bad Request, 401 Unauthorized, 402 Payment Required, 403 Forbidden, 404 Not
Found, 405 Method Not Allowed, 406 Not Acceptable, 407 Proxy Authentication
Required, 408 Request Timeout, 409 Conflict, 410 Gone, 411 Length Required, 412
Precondition Failed, 413 Payload Too Large, 414 URI Too Long, 415 Unsupported
Media Type, 416 Range Not Satisfiable, 417 Expectation Failed, 418 I’m a Teapot,
421 Misdirected Request, 422 Unprocessable Entity, 423 Locked, 424 Failed
Dependency, 425 Too Early, 426 Upgrade Required, 428 Precondition Required, 429
Too Many Requests, 431 Request Header Fields Too Large, 451 Unavailable For
Legal Reasons

### 5xx Server Errors

500 Internal Server Error, 501 Not Implemented, 502 Bad Gateway, 503 Service
Unavailable, 504 Gateway Timeout, 505 HTTP Version Not Supported, 506 Variant
Also Negotiates, 507 Insufficient Storage, 508 Loop Detected, 510 Not Extended,
511 Network Authentication Required
