---
title: Caching
---

## Caching Strategies in Web Applications and Microservices

Caching strategies determine how and when the cache is populated, read, and
updated in relation to the primary data store (such as a database or a remote
API).

### Write-Through Caching

Write-through caching synchronously updates both the cache and the underlying
database for every write operation. Each data change is written simultaneously
to both layers, guaranteeing that the cache and persistent storage are always
consistent.

**Strengths**:

- _Consistency_: The cache always reflects the latest database state.
- _Recovery_: Safe recovery from cache failures—data is always present in the
  database.

**Weaknesses**:

- _Latency_: Every write operation is slower due to the overhead of two writes.
- _Write Amplification_: Increased load on primary storage, potentially negating
  some cache advantages for write-heavy workloads.

**Use Cases**: Write-through is ideal when strong consistency is required and
write frequency is moderate compared to reads—for example, in online banking
transactions or order processing systems.

### Write-Back (Write-Behind) Caching

Write-back (or write-behind) caching acknowledges a write operation as soon as
it is stored in the cache but writes (flushes) the change to the underlying
database asynchronously, typically upon cache eviction or after a fixed delay.

**Strengths**:

- _Performance_: Fast write responses by deferring slow database updates.
- _Reduced Write Load_: Can batch or coalesce writes, reducing database I/O.

**Weaknesses**:

- _Risk of Data Loss_: If the cache fails before data is flushed, updates might
  be lost.
- _Consistency Complexity_: Data in the database may lag behind the cache,
  leading to potential stale data for consumers reading directly from the
  database.

**Use Cases**: Write-back is preferred for write-intensive applications where
some risk of data loss is acceptable and performance is paramount—such as
logging, analytics, or non-critical state recording.

### Write-Around Caching

Write-around caching bypasses the cache on every write operation, directing the
data exclusively to the database. The cache is only updated on subsequent reads
if there is a cache miss; data flows into the cache as it is accessed (i.e.,
lazily loaded).

**Strengths**:

- _Prevents Cache Pollution_: Particularly effective for infrequently accessed
  or write-only data that would waste valuable cache space.
- _Simple Consistency_: The database remains the source of truth.

**Weaknesses**:

- _Initial Read Misses_: Recently written data will incur cache misses upon
  first read, causing higher latency on first access.
- _Staleness_: If not coupled with a cache invalidation policy, there is a risk
  of reading stale data (old value still in cache after a write).

**Use Cases**: Write-around excels in applications dominated by write-heavy or
bulk-insert access patterns, especially where future reads are rare—such as
event logs, queue messages, or archival data.

### Read-Through / Cache-Aside (Lazy Loading) Strategy

In read-through caching, the application requests data exclusively from the
cache. On a cache miss, the cache system retrieves the data from the backing
store, stores it in the cache, and returns it to the application. In cache-aside
(lazy loading), the application is responsible for checking the cache upon every
read and updating it on a miss.

**Strengths**:

- _Flexible Lazy Population_: Only populates data that is actually accessed;
  avoids wasting memory on cold data.
- _Resilience_: If the cache fails, the application can always fall back to the
  backing store.

**Weaknesses**:

- _Initial Latency_: First-time requests for a given item experience cache
  misses.
- _Consistency_: May necessitate active invalidation mechanisms to ensure
  synchronization with the underlying data store.

**Use Cases**: This is the default pattern for most distributed caching
deployments (e.g., Memcached, Redis, Hazelcast). It is particularly effective
for read-heavy, bursty, or unpredictable workloads, like user profile lookups,
product catalogs, or dynamic web content.

### Advanced and Hybrid Strategies

**Refresh-Ahead** and **Stale-While-Revalidate** go further by proactively
refreshing cache entries before expiry, or allowing the cache to serve stale
data while asynchronously fetching fresh versions in the background. These
advanced strategies are particularly useful for hiding cache misses and
smoothing sudden traffic spikes (“cache stampedes”).

---

#### Strategy Summary Table

| Strategy               | Write Path               | Read Path                 | Consistency | Latency      | Best For                                  |
| ---------------------- | ------------------------ | ------------------------- | ----------- | ------------ | ----------------------------------------- |
| Write-Through          | Cache + DB sync          | Cache                     | Strong      | High Write   | Strong consistency, moderate write load   |
| Write-Back             | Cache, async flush to DB | Cache                     | Eventual    | Low Write    | Write-heavy, tolerable risk of data loss  |
| Write-Around           | DB only                  | Cache (if present) or DB  | Strong      | Varies       | Bulk writes, infrequently read data       |
| Read-Through           | N/A                      | Cache, fallback to DB     | Varies      | Low (on hit) | Read-heavy, unpredictable workloads       |
| Cache-Aside            | N/A                      | App handles cache+DB      | Varies      | Low (on hit) | Custom manipulation, fine-grained control |
| Refresh-Ahead          | N/A (background writes)  | Cache, background refresh | Varies      | Ultra-Low    | Hot-spot data, high staleness tolerance   |
| Stale-While-Revalidate | N/A                      | Serve stale, background   | Soft/Ev.    | Ultra-Low    | Preventing stampedes, reliability favors  |

---

## Cache Eviction Policies

Cache eviction policies determine which data should be discarded from the cache
when it reaches its maximum capacity or when data becomes stale. The chosen
eviction policy has substantial impact on cache effectiveness, hit rates, and
overall system performance, making it a fundamental design choice.

### Least Recently Used (LRU)

**Principle**: Evict the data item that has not been accessed for the longest
period. LRU works under the heuristic that recently used items are more likely
to be needed again soon.

**Implementation**:

- Doubly linked lists and hash maps to track access order while maintaining O(1)
  access.
- Approximate variants (e.g., clock algorithm) used for memory efficiency.

**Pros**:

- Intuitive; matches temporal locality in real-world workloads.
- Good for web caching, user sessions, database buffer management.

**Cons**:

- Overhead for recency tracking.
- May not perform optimally with certain cyclic access patterns
  (“ping-pong”/round-robin).

**Use Cases**:

- Web content/CDN caches.
- In-memory database caching.
- Filesystem directory/file metadata caches.

### Least Frequently Used (LFU)

**Principle**: Remove the cache entry with the lowest frequency of access over
time.

**Implementation**:

- Hash maps and frequency lists to track access counts.
- O(1) LFU achieved by grouping keys by count (“frequency nodes”).
- Can be augmented with aging to mitigate old entries “sticking” in cache.

**Pros**:

- Well-suited for workloads with consistent, periodic access (e.g., product
  pages, popular articles).

**Cons**:

- Can retain stale data if frequency isn’t aged/flushed.
- More complex to implement (frequent counter updates, multiple data
  structures).

**Use Cases**:

- CDN/proxy object caches.
- Network routing table caches.
- Personalized recommendation engines.

### First-In-First-Out (FIFO)

**Principle**: Evict the oldest added element, regardless of access history.

**Implementation**:

- Often implemented as a simple queue.

**Pros**:

- Easy to implement.
- Low overhead, deterministic.

**Cons**:

- Ignores actual usage pattern; high risk of removing “hot” items that were just
  accessed.

**Use Cases**:

- Message queue management.
- Sequential data streaming where strict order is required.

### Random Replacement

**Principle**: Evict a randomly chosen entry when the cache is full.

**Pros**:

- Simplest and lowest overhead.
- Avoids tracking history.

**Cons**:

- Evicts important items unpredictably; typically worst miss rate in production.

**Use Cases**:

- Non-critical caches, testing, resource-constrained environments.

### Time-Based Expiry (TTL)

**Principle**: Assign each item a fixed Time-To-Live (TTL); expire items after
this interval.

**Implementation**:

- Scheduled or background cleanup.
- Used alongside other schemes for database cache freshness.

**Pros**:

- Ensures cache doesn’t serve outdated data.
- Great for ephemeral/session data and time-bounded logic.

**Cons**:

- Irrespective of usage pattern.
- Selecting proper TTL is often non-trivial—too long: staleness; too short: low
  hit rate.

**Use Cases**:

- User sessions, authentication tokens.
- CDN caches (per HTTP Cache-Control).

### ARC (Adaptive Replacement Cache), 2Q, and Hybrid Policies

**ARC**: Balances recency and frequency by keeping separate lists for “recently
used” and “frequently used” items, adapting sizes automatically. Seen in ZFS,
IBM hardware, VMware vSAN.

**2Q**: Maintains separate queues for recent vs frequent access; reduces cache
pollution by one-hit-wonder items.

**Other Advanced Schemes**: CLOCK, S3-FIFO, SLFU, LFRU, LIRS, etc.—used in
database systems, OS kernels, or high-volume web caches to maximize hit rates
under challenging workloads.

---

#### Eviction Policy Comparison

| Policy | Criteria         | Memory Overhead | Adaptability | Pros / Cons                                                                          | Typical Use Cases                     |
| ------ | ---------------- | --------------- | ------------ | ------------------------------------------------------------------------------------ | ------------------------------------- |
| LRU    | Least recent use | Moderate        | High         | Easy, intuitive; requires tracking; time-based; poor when “cold”/cyclic access       | Web, DB, file system, CDN             |
| LFU    | Least freq. use  | High            | Moderate     | Tracks heavy hitters; keeps stable data; complex to implement; may hold on to “dead” | Recommendations, DBs, APIs            |
| FIFO   | Insertion order  | Low             | None         | Simple, predictable; ignores recency/frequency                                       | Queues, streaming, OS tasks           |
| Random | None             | Very Low        | None         | Simple; suboptimal hit rates; risky for prod                                         | Testing, constrained hardware         |
| TTL    | Age              | Low             | None         | Freshness guarantees; TTL tuning required; not usage-aware                           | Sessions, tokens, HTTP/CDN            |
| ARC/2Q | Adaptive         | High            | High         | Handles mixed workloads; complex implementation                                      | DBs, OS page caches; ZFS, VMware vSAN |

---

## Cache Types by System Layer

Caching can be applied at nearly every architectural layer of a modern system,
with each layer serving unique functional and performance goals. Below is an
analysis of the most commonly used cache types, aligned to specific layers.

### Cache Types by System Layer

| Layer             | Cache Type                      | Description / Examples                                   | Tooling/Technologies                                                        |
| ----------------- | ------------------------------- | -------------------------------------------------------- | --------------------------------------------------------------------------- |
| **Database**      | Query cache, Result set cache,  | Stores frequent queries, joins, or materialized views    | MySQL Query Cache, PostgreSQL, Redis, Memcached                             |
|                   | Write buffer, Materialized view | for reduced load and fast analytics                      | Oracle Result Cache, DynamoDB DAX                                           |
| **Backend / App** | In-memory, Distributed, Near-   | Stores application objects, API responses, business data | Redis, Memcached, Hazelcast, Guava                                          |
|                   | cache, Replicated, Local        | for speed; may be shared or per-instance                 | Spring Cache, Caffeine, .NET IMemoryCache                                   |
| **Load Balancer** | Reverse Proxy, CDN, Edge cache  | Stores HTTP responses, static content, API keys, cookies | Nginx, Varnish, Cloudflare, Akamai, Fastly                                  |
| **Front-End**     | Browser cache, Service workers  | Stores static assets, computed data, API responses       | HTTP headers, Cache-Control, ETag, IndexedDB, localStorage, Service Workers |

---

### Database-Level Caching

- **Query-Level Caches:** Store specific SQL query results, reducing
  computational load for repeated, expensive queries.
- **Result Set/Materialized View:** Precompute and store complex query
  joins/aggregations, favored in analytics- and reporting-heavy environments.
- **Write Buffers:** Temporarily store pending database writes; periodically
  flushed for eventual consistency.
- **In-Memory Data Grids/IMDBs:** E.g., Redis, Hazelcast for live, high-volume,
  low-latency database workloads.

### Backend/Application-Level Caching

- **In-Memory Local Cache:** Stores session data, auth tokens, or small "hot"
  working sets per app instance—e.g., Java’s Guava, .NET `MemoryCache`.
- **Distributed Cache Cluster:** A shared cache spanning multiple service
  nodes—e.g. Redis Cluster, Amazon ElastiCache, Hazelcast, or
  Memcached—facilitating scale-out consistency and resilience.
- **Replicated / Near-Cache:** Combines fast, local per-instance cache with
  centralized coherence, blending speed and accuracy.

### Load Balancer / CDN and Reverse Proxy Caching

- **Edge/Proxy Caches** (CDN): Store static content close to users for global
  scale—images, CSS, JS, videos.
- **HTTP Reverse Proxies:** Offload dynamic and personalized content closer to
  the user, intercepting requests at the edge or in front of the web server.
- **TTL, ETag, and `Cache-Control`** headers play a crucial role in managing
  content staleness, privacy, and validation.

### Front-End/Browser Caching

- **Local Storage:** HTML5 APIs (localStorage, IndexedDB) for persistent
  client-side data.
- **Browser Cache:** Resistant to server interruptions; stores static assets
  under the guidance of HTTP cache headers (`Cache-Control`, `ETag`, `Expires`,
  etc.).
- **Service Workers:** Enable offline-first experiences by intercepting network
  requests for custom cache logic at the browser level.

---

## Cache Concurrency Strategies

| Strategy             | Consistency            | Use Case                                           | Notes                                            |
| -------------------- | ---------------------- | -------------------------------------------------- | ------------------------------------------------ |
| ReadOnly             | Strong (never changes) | Reference data, static lookup tables               | Exception on update; safest and most performant  |
| NonStrict Read/Write | Eventual               | Data updated infrequently; rare conflicts          | May serve stale data; update on commit only      |
| Read/Write           | Strong (soft-locks)    | Frequently updated data needing consistency        | Uses soft locks, not SERIALIZABLE; more overhead |
| Transactional        | XA-transactional       | Data needing distributed transactional consistency | Used with transactional caches and JTA           |

- **ReadOnly**: Cache is immutable after the initial load. Any update attempts
  throw exceptions. This is optimal for static metadata and achieves the highest
  performance and safety in both standalone and clustered environments.
- **Non-Strict Read/Write**: Data may be manually or lazily invalidated after
  updates, with no guarantee of immediate consistency, tolerating short-lived
  inconsistencies for infrequently modified data.
- **Read/Write**: Soft locks manage updates; cache is invalidated or updated
  after transactions commit, offering strong consistency without full isolation.
- **Transactional**: Cache updates participate in distributed transactions,
  offering the strongest consistency guarantees, but with considerable
  complexity and overhead. This pattern is only viable where durable
  transactional consistency is paramount, such as financial systems with
  cluster-aware cache providers.

---

## Distributed Locking

Maintains mutual exclusion in distributed environments. Faulty implementations
(e.g., Redis Redlock) may fail under network partitions or GC-induced process
pauses. For correctness, distributed locks should support _fencing tokens_,
i.e., monotonically increasing numbers that validate the epoch of lock
acquisition—ensured by consensus protocols like ZooKeeper, Raft, or Paxos.
Efficiency-driven (rather than correctness-driven) locks may use simple Redis or
advisory DB locks. The absence of fencing can lead to data corruption or
split-brain writes.

---

## Microservices Caching Patterns and Topologies

Microservices introduce unique caching challenges: each service owns its own
datastore (bounded context), but frequently interacts with data from neighboring
services. Caching in microservices must navigate a balance between performance,
consistency, autonomy, and data stewardship.

### Common Patterns

1. **Single In-Memory Cache**: Each service instance maintains its own cache.
   Suitable for small datasets (config, enum values) but susceptible to cache
   incoherence if instances scale up or down.

2. **Distributed Client-Server Cache**: A central cache cluster (e.g.,
   Redis/Memcached) is shared by all microservice instances, offering fast,
   consistent reads, failover, and scaling.

3. **Replicated/Hybrid Cache (Near-Cache)**: Combines a fast, per-instance local
   cache for "hot" keys with a central cache for full consistency, often using
   versioning or push-based invalidation.

4. **Sidecar Pattern**: In Kubernetes, cache is deployed as a sidecar container
   alongside each microservice, providing cache locality with cross-instance
   coherence (local communication via localhost).

5. **Reverse Proxy Pattern**: HTTP reverse proxy (e.g., Varnish, Nginx) sits in
   front of the service tier, handling cacheable requests before reaching the
   service.

6. **Data Sharing and Data Sidecar**: Ownership-respecting patterns where
   read-only consumers cache external data with periodic refreshes, but never
   write or invalidate another service’s store.

---

#### Microservices-Specific Concerns

- **Bounded Context**: Each service should cache only data it owns, or rely on
  event-driven sidecars/refreshes to propagate state from owners—never bypass
  direct service APIs to the data layer of neighbors.
- **Stale Data and Eventual Consistency**: Accepting a window of staleness is
  often required for high throughput; careful invalidation/event notification or
  short TTLs help balance freshness and cost.
- **Topology Selection**: Choose between single-instance, distributed,
  near-cache, or hybrid based on:
  - State size
  - Update frequency
  - Consistency requirements
  - Fault tolerance/resilience mandate

---

### Microservices Caching Topology Comparison

| Topology                 | Pros                                     | Cons                                            | Best Fit                                |
| ------------------------ | ---------------------------------------- | ----------------------------------------------- | --------------------------------------- |
| In-Memory (Per Instance) | Ultra-fast, easy config, no ops overhead | No cross-instance sync, prone to incoherence    | Tiny static sets, local configs         |
| Distributed Cluster      | High consistency, scalable, reliable     | Network latency, ops complexity                 | Shared session state, high-volume data  |
| Near-Cache Hybrid        | Combines speed+consistency               | Complexity (invalidation, versioning needed)    | Product catalogs, reference lookups     |
| Sidecar                  | Local speed, generic pod config          | Only works with containerized, collocated cache | Microservices/Kubernetes clusters       |
| Reverse Proxy            | No app change, language agnostic         | Limited to HTTP/REST, invalidation tricky       | Web APIs, static/dynamic asset fronting |

---

## Popular Caching Tools & Technologies

- **Redis**: In-memory key-value store supporting rich data types, atomic
  operations, TTLs, clustering, and replication. Popular for sessions,
  leaderboards, distributed locks.
- **Memcached**: Lightweight, high-throughput key-value cache, excellent for
  ephemeral or transient data, no built-in persistence.
- **Hazelcast/Infinispan/Apache Ignite**: Distributed JVM data grids suitable
  for Java and polyglot workloads, support advanced topologies (IMDG, IMDB,
  near-cache).
- **Varnish**: Edge-optimized HTTP reverse proxy cache, supports custom policies
  for media-heavy and high-traffic websites.
- **CDNs** (Akamai, Cloudflare, Fastly): Geographically distributed edge caching
  for static content and web acceleration.

---

## Interview Questions

### Define caching and explain its advantages in system architecture.

Caching is the process of storing copies of data in fast-access storage (memory,
disk, or distributed clusters) to serve future requests faster than fetching
from slower, primary sources like databases or remote APIs. Caching improves
system performance by reducing retrieval latency, minimizing redundant
computations, lowering database/backend load, increasing throughput (especially
for read-dominant traffic), and boosting fault tolerance by serving requests
locally during backend outages.

### What’s the difference between a cache hit and a cache miss?

A cache hit occurs when the requested data is found in the cache and returned
immediately. A cache miss happens when the requested data is absent from the
cache, requiring a fetch from the slower backing source, which may be
subsequently cached for future access.

### Describe key differences and trade-offs between write-through, write-back, and write-around caching.

- _Write-through_: All writes are persisted synchronously to both cache and
  database, ensuring strong consistency but increasing write latency.
- _Write-back_: Writes are acknowledged immediately (cache only) and flushed to
  database asynchronously, improving speed but risking data loss on failure.
- _Write-around_: Writes bypass cache, updating only the database; cache is
  updated lazily upon future reads, which reduces cache pollution but may cause
  initial cache misses and temporary staleness.

### Describe LRU and LFU. When would you favor one over the other?

- _LRU (Least Recently Used)_: Evicts the least recently accessed entry—best for
  workloads with temporal locality (e.g., trending news).
- _LFU (Least Frequently Used)_: Evicts least often accessed entry—ideal for
  stable, long-term access frequency (e.g., evergreen product pages or
  configuration keys).  
  If rapid access patterns change, LRU adapts more quickly; if popularity is
  stable over time, LFU achieves better hit ratios.

### What is a “cache stampede”? How can it be prevented?

Cache stampede describes the scenario where many threads/processes
simultaneously encounter a cache miss and recompute or fetch the same expensive
data, causing backend overload.  
Mitigation includes locking/serialization (e.g., mutex or single-flight),
serving stale data while refreshing, probabilistic early TTL expiration, or
request deduplication.

### What does the TTL parameter do?

TTL (Time-To-Live) defines the expiration duration for a cache entry. Once TTL
elapses, the entry is evicted and subsequent reads will fetch fresh data. TTL
balances data freshness and cache efficiency.  
Short TTLs minimize staleness but decrease hit ratio; long TTLs maximize
performance but may risk serving outdated information.

### Compare and contrast the strengths of FIFO and Random eviction compared to LRU and LFU.

FIFO is straightforward and memory-efficient—evicts by insertion order, no
access history tracked. Useful for streaming/batched data. Random eviction is
even simpler but often delivers worst-case performance for hit ratios.  
Both are easy to implement but inflexible; LRU/LFU better capture real access
patterns and importance.

### How does distributed caching work, and what are its challenges?

Distributed caching shares cache data across multiple nodes/servers,
synchronizing state for resilience and horizontal scaling.  
Challenges: ensuring cache coherence/invalidation across nodes, split-brain
(inconsistent state due to network partitions), replication delays, and tuning
sharding/partitioning for optimal key distribution and load balancing.

### Why is cache invalidation called one of the “two hard things”?

Properly expiring or refreshing cached data upon updates is difficult—due to
race conditions, network delays, or multiple representations of the same
content. Incorrect invalidation results in stale data, bugs, or downtime. In
microservices, changes to one service may impact cached content in others,
demanding event-driven approaches and precise invalidation policies.

### Explain the role of edge (CDN) caching and browser caching.

CDN caching stores static content (images, JS, CSS, API responses) at
geographically distributed points of presence close to users, dramatically
reducing latency, bandwidth, and server load.  
Browser caching leverages local browser memory/disk to persist assets or data,
controlled by HTTP headers (`Cache-Control`, `ETag`, `Expires`). These are
first-level defenses for repeat resource loading in web applications.

### When is it preferable to cache at the application layer versus the database?

Application-layer caches handle processed or computed data and can embed
domain-specific caching logic (memoization, partial view caching). They offer
more flexible, business-aware management and can combine multiple data sources.  
Database-layer caches are transparent to the application, offload frequent read
queries from the DB, but may cache unused (“cold”) data lacking domain context.
Use application-layer caching for complex or join-rich business logic, and
data-layer caching for pure query acceleration.

### What’s the best way to tune the cache size and eviction policy in production?

Monitor the cache hit/miss ratio, memory usage, eviction rates, and response
latency. Begin with LRU/LFU and adjust TTL or cache size iteratively based on
real-world metrics. If hit ratio stagnates or miss rates spike, consider hybrid
or workload-specific policies. Continuous tuning and real-time analytics inform
optimal configuration.

### How do you design cache keys in microservices for maximum efficiency and safety?

Keys must be unique, meaningful, and compact. Use namespacing: e.g.,
`user:{user_id}:settings`. Include versioning or timestamps to force busts on
major schema changes. Omit personal/private data to prevent security leaks
across multi-tenant boundaries. Favor string or binary keys compatible with all
client languages.

### What are the consistency trade-offs in distributed cache invalidation?

- Strong consistency (synchronous invalidation): Ensures no stale reads but may
  slow the system and increase network chatter.
- Eventual consistency (asynchronous, event-driven): Offers high throughput and
  resilience but tolerates small windows of data staleness. Use
  publish-subscribe invalidation, TTL alignment, or event sourcing/integration
  layers to coordinate distributed cache expiry without global locks.
