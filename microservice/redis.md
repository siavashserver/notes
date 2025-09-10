---
title: Redis (Remote Dictionary Server)
---

It's an in-memory data‑structure store used for cache, session, messaging, etc.,
offering low-latency reads/writes.

## Data Types

- String: Binary-safe value, up to 512 MB
- Hash: Field/value maps, ideal for modeling objects
- List: Ordered lists; push/pop operations from both ends
- Set: Unordered unique collections; supports unions and intersections
- Sorted Set (ZSET): Unique strings with scores, ordered by score

```csharp
using StackExchange.Redis;

class RedisDemo {
  static void Main() {
    var mux = ConnectionMultiplexer.Connect("localhost:6379");
    var db = mux.GetDatabase();

    // String
    db.StringSet("key:string", "Hello Redis!");
    string str = db.StringGet("key:string");
    Console.WriteLine(str);  // Hello Redis!

    // Hash
    db.HashSet("hash:user:1", new HashEntry[] {
        new("name", "Alice"), new("age", "30")
    });
    var name = db.HashGet("hash:user:1", "name");
    Console.WriteLine(name);  // Alice

    // List
    db.ListLeftPush("list:tasks", "task1");
    db.ListRightPush("list:tasks", "task2");
    RedisValue[] tasks = db.ListRange("list:tasks", 0, -1);
    Console.WriteLine(string.Join(", ", tasks));  // task1, task2

    // Set
    db.SetAdd("set:colors", "red");
    db.SetAdd("set:colors", "blue");
    bool isMember = db.SetContains("set:colors", "red");
    Console.WriteLine(isMember);  // True

    // Sorted set
    db.SortedSetAdd("zset:scores", "player1", 100);
    db.SortedSetAdd("zset:scores", "player2", 150);
    var top = db.SortedSetRangeByScoreWithScores("zset:scores", order: Order.Descending);
    foreach(var entry in top)
        Console.WriteLine($"{entry.Element}: {entry.Score}");

    mux.Close();
  }
}
```

## Time-To-Live (TTL)

```csharp
var conn = ConnectionMultiplexer.Connect("localhost");
var db = conn.GetDatabase();

// String with expiry
db.StringSet("session:123", "user-data", TimeSpan.FromMinutes(30));

// Alternatively for any type:
db.KeyExpire("user:123", TimeSpan.FromHours(1));

// Check remaining TTL
TimeSpan? ttl = db.KeyTimeToLive("user:123");
Console.WriteLine(ttl.HasValue ? ttl.Value.ToString() : "no TTL");

// Remove TTL
db.KeyPersist("user:123");
```

## Eviction Policies

Redis eviction policies determine what happens when Redis reaches its
`maxmemory` limit.

- `allkeys-` -> includes all keys regardless of TTL
- `volatile-` -> only considers keys with TTLs

- `noeviction`: Rejects new writes once memory is full; no keys are evicted
- `allkeys-lru`: Evicts the least recently used across all keys
- `volatile-lru`: Same LRU logic, but only on keys with a TTL set (**default**)
- `allkeys-lfu`: Evicts least frequently used across all keys
- `volatile-lfu`: LFU only on TTL-backed keys
- `allkeys-random`: Randomly evicts any key
- `volatile-random`: Random eviction among TTL keys
- `volatile-ttl`: Evicts TTL keys with the _shortest remaining time_

```csharp
using StackExchange.Redis;

// Setup connection
var mux = ConnectionMultiplexer.Connect("localhost:6379");
var server = mux.GetServer("localhost", 6379);

// Set memory limit (e.g., 100 MB) and eviction policy
server.ConfigSet("maxmemory", "100mb");
server.ConfigSet("maxmemory-policy", "allkeys-lru");
```

## Setting Up Master-Slave Replication

- Install Redis on master and slave nodes
- On the slave, set in `redis.conf`: `replicaof <master-ip> 6379`

## Master-Slave Synchronization Methods

- Full sync: initial sync sends the full dataset to slaves when they join
- Partial sync (PSYNC): after initial sync, only incremental commands are sent
- By default, replication is asynchronous: the master doesn't wait for slaves to
  acknowledge writes. You can request some level of sync using the `WAIT`
  command

---

## Redis as a Distributed Lock Manager

Redis has become an industry-standard tool for distributed locking due to its
strengths as an in-memory, single-threaded, high-performance data store that
supports atomic commands. Its key attributes make it well-suited for
coordination tasks:

- **Atomic primitives**: Operations such as SET with unique options (NX, PX) and
  SETNX are inherently atomic.
- **High throughput and low latency**: Ideal for lock acquisition and release
  under load.
- **Persistence and replication options**: Suitable for both volatile and more
  durable lock semantics.
- **Wide client library support**: Including robust support in .NET/C#, Java,
  Python, Go, etc.
- **Horizontal scalability**: Redis can be deployed as single instances,
  master-slave, or with Sentinel/Cluster, enabling advanced distributed lock
  algorithms such as Redlock.

However, it’s equally vital to understand the operational boundaries and
caveats: Redis, when used in a distributed lock context, is not a transactional
database, and its consistency guarantees are predicated on deployment topology,
failover configuration, and clock drift handling. This forms the core of the
transition from naive locking with **SETNX** to the more sophisticated,
multi-node **Redlock algorithm**.

---

## Redis SETNX Command for Basic Locking

The simplest locking mechanism in Redis is the **SETNX** (Set if Not Exists)
command, which performs an atomic "check-and-set":

```shell
SETNX lock:key unique_value
```

If the key does not exist, it is set (lock acquired); if the key already exists,
the operation fails (lock not acquired by this client). This enables a
rudimentary lock acquisition pattern—the process that sets the lock can perform
its critical section, then delete the key to release the lock:

```plain
SETNX mylock 1234
# ...perform critical work...
DEL mylock
```

**C# Example with StackExchange.Redis**:

```csharp
var db = redis.GetDatabase();
bool acquired = await db.StringSetAsync("mylock", uniqueValue, TimeSpan.FromSeconds(30), When.NotExists);
if (acquired)
{
    // Perform critical task
    await db.KeyDeleteAsync("mylock");
}
```

This approach works for simple, single-node Redis deployments and can be
effective for noncritical coordination needs.

#### Limitations of SETNX for Locking

1. **No automatic expiration**: A lock key created with SETNX is held until
   explicitly deleted, which poses danger if the process holding the lock
   crashes or the network partitions—other clients might be blocked forever
   (deadlock).
2. **No lock ownership**: Since SETNX sets only the value, one instance could
   inadvertently delete another's lock (if not using unique values for different
   instances).
3. **Singleton safety**: Only safe in single-instance Redis mode; if using
   master/slave, failover, or clusters, a write may be acknowledged by a master
   node even if slaves have not yet synchronized, risking split-brain when
   failover occurs.

To mitigate these, best practice recommends the use of an expiration time (via
the EX/PX options) and a unique identifier per lock acquisition.

---

## Redis Key Expiration and Auto-Release

**Key expiration** is a safety net for distributed locks. By associating a
time-to-live (TTL) with the lock key, Redis automatically expires (deletes) the
lock after a specified interval, reducing the risk of deadlocks due to process
crashes, network failures, or code bugs:

```shell
SET mylock unique_value NX PX 30000
```

- `NX` ensures the key is only set if it does not exist (i.e., lock is only
  granted if not held).
- `PX` sets the expiration in milliseconds (so, 30,000 ms or 30 seconds here).

**Auto-release** is crucial in production; without it, transient failures could
leave the system inoperable. However, setting TTLs introduces its own
trade-offs:

- If a process takes longer than the TTL to complete its critical section,
  another process may reacquire the lock and both process concurrently—violating
  mutual exclusion.
- Too short an expiration reduces deadlock risk but increases the risk of
  split-brain access; too long, and loops in recovery are sluggish.

**Robust Implementation Additions:**

- Use a unique lock value (typically a UUID) so that only the lock owner can
  release the lock, e.g., by scripting the unlock operation to ensure that only
  the holder’s unique value can delete the lock.
- Use Lua scripts in Redis to perform atomic “check-and-delete” unlock commands.

**C# Example**:

```csharp
// Acquire lock with expiry
bool acquired = await db.StringSetAsync("lock:myresource", lockValue, TimeSpan.FromSeconds(30), When.NotExists);

// Release lock only if lock value matches
string script = @"
    if redis.call('get', KEYS[1]) == ARGV[1] then
        return redis.call('del', KEYS[1])
    else
        return 0
    end";
bool released = (int)await db.ScriptEvaluateAsync(script, new RedisKey[] { "lock:myresource" }, new RedisValue[] { lockValue }) == 1;
```

This pattern ensures that locks are auto-expired and only the acquiring client
can release its own lock, significantly reducing the risk of accidental unlocks.

---

## Redlock Algorithm: Design and Guarantees

### Introduction to Redlock

Due to the well-documented shortcomings of single-instance locking (data loss,
split-brain, race conditions post-failover), Redis' creators proposed
**Redlock**, a distributed lock algorithm specifically designed to work across
multiple independent Redis instances. The Redlock protocol achieves strong
safety guarantees even in the presence of network partitions, clock drift, and
node failures.

> In Redis, **split-brain risk** refers to a dangerous situation where two or
> more nodes in a distributed setup—like a master and its
> replicas—_simultaneously believe they are the master_. To prevent it, in
> Sentinel mode, set a proper quorum so failover only happens if a majority
> agrees the master is down.

#### Redlock Algorithm Steps

The steps below summarize the Redlock lock acquisition flow:

1. **Generate a unique lock value** (e.g., UUID) per client/process.
2. **Request the lock in parallel** from at least N Redis instances (N is
   usually 5 for good safety; must be odd), using SET resource_name unique_value
   NX PX lock_ttl.
3. **Count the number of successful acquisitions** (i.e., those where Redis
   returns OK).
4. **Prove lock validity**: If the majority of the instances (at least N/2 + 1)
   have granted the lock, and the total time taken to request all locks is less
   than the lock's TTL, then lock is acquired.
5. **If acquisition fails**, release any acquired locks immediately to avoid
   partial acquisition.
6. **Release the lock**: Delete the lock key from all instances, but only if the
   lock value matches (to avoid unlocking someone else's lock).

### Redlock Guarantees

Redlock offers the following _safety_ and _liveness_ properties:

- **Mutual exclusion**: At most one client can hold the lock at any given time.
- **Deadlock freedom**: It is always possible to acquire the lock eventually,
  even if some clients crash or the network partitions.
- **Fault tolerance**: Clients can acquire and release locks as long as the
  majority of Redis nodes are reachable and up-to-date.

#### Design Considerations

Redlock's design is predicated on several assumptions for fault tolerance:

- Redis instances should _not_ share a common master or data store—each instance
  must be independent to avoid correlated failures.
- Network delays, partitions, and instance failures must be considered. Redlock
  is only as strong, in terms of correctness, as the independence and
  availability of the majority of its Redis nodes.
- The TTL should be chosen to accommodate reasonable network latency and
  critical section execution time.

---

## Single Redis Instance vs. Redlock Multi-Instance

| Feature               | Single Redis (SETNX/NX+EX)              | Redlock (Multi-instance)                                          |
| --------------------- | --------------------------------------- | ----------------------------------------------------------------- |
| Mutual Exclusion      | Only as reliable as one node            | At least majority reliability                                     |
| Partition Tolerance   | Fails if master fails or is partitioned | Survives node/network failures, as long as most nodes are healthy |
| Data Loss After Crash | Possible (risk of orphaned locks)       | Much reduced, thanks to quorum                                    |
| Split-Brain Risk      | High on failover                        | Lower—requires majority failure                                   |
| Complexity            | Simpler to set up and operate           | More complex (multiple Redis servers)                             |
| Performance           | Slightly higher, due to no quorum check | Slight overhead, but safer                                        |
| Usage Scenario        | Non-critical, single-server             | Mission-critical, highly available locking                        |

While single-instance Redis locks (using SETNX) suffice for simple scenarios
where failover, durability, and correctness are less problematic, Redlock is
strongly recommended for high-reliability production systems requiring robust
distributed coordination under network partitions or node failures.

---

## C# Implementation of Redis-Based Locking

### StackExchange.Redis for Locking

The [StackExchange.Redis](https://github.com/StackExchange/StackExchange.Redis/)
library is the de facto Redis client for .NET and C# applications. It provides
async APIs and supports all Redis primitives. For locking, it exposes methods
such as:

- `LockTake` (acquire lock)
- `LockRelease` (release lock)

However, by default, these are wrappers over SETNX and DEL, making them suitable
mostly for single-node usage. Developers must implement Lua script-based
check-and-delete for safe lock ownership verification.

#### Example: Basic Lock Implementation Using StackExchange.Redis

```csharp
using StackExchange.Redis;
using System;

public class RedisLock
{
    private readonly IDatabase _db;
    private readonly string _lockKey;
    private readonly string _lockValue;
    private readonly TimeSpan _expiry;

    public RedisLock(IDatabase db, string key, TimeSpan expiry)
    {
        _db = db;
        _lockKey = key;
        _lockValue = Guid.NewGuid().ToString();
        _expiry = expiry;
    }

    public async Task<bool> AcquireAsync()
    {
        return await _db.StringSetAsync(_lockKey, _lockValue, _expiry, When.NotExists);
    }

    public async Task<bool> ReleaseAsync()
    {
        // Use Lua script for atomic release
        string script = @"
            if redis.call('get', KEYS[1]) == ARGV[1]
            then
                return redis.call('del', KEYS[1])
            else
                return 0
            end";

        var result = (int)await _db.ScriptEvaluateAsync(script, new RedisKey[] { _lockKey }, new RedisValue[] { _lockValue });
        return result == 1;
    }
}
```

This wrapper ensures only the owner releases the lock. It is best practice to
associate each lock instance with a unique value.

---

### RedLock.net for Multi-Instance Redlock

[RedLock.net](https://github.com/samcook/RedLock.net) is a mature,
NuGet-published C# implementation of the Redlock distributed lock algorithm. It
leverages StackExchange.Redis, handling the complexity of quorum-based
acquisition and automatic renewal (prolongation) as well as proper release
semantics.

#### Setup and Usage Example:

```csharp
using RedLockNet;
using RedLockNet.SERedis;
using RedLockNet.SERedis.Configuration;
using System.Collections.Generic;
using System.Net;

// Define Redis endpoints (should be independent Redis servers)
var endpoints = new List<RedLockEndPoint> {
    new DnsEndPoint("redis1", 6379),
    new DnsEndPoint("redis2", 6379),
    new DnsEndPoint("redis3", 6379)
};

// Create RedLock factory
var redlockFactory = RedLockFactory.Create(endpoints);

TimeSpan expiry = TimeSpan.FromSeconds(30);
TimeSpan wait = TimeSpan.FromSeconds(10);
TimeSpan retry = TimeSpan.FromMilliseconds(250);

using (var redLock = await redlockFactory.CreateLockAsync(
    "my_critical_resource", expiry, wait, retry))
{
    if (redLock.IsAcquired)
    {
        // Critical section
    }
    // Lock auto-released at the end of using
}
```

**Explanation**:

- `expiry`: Maximum time to hold the lock.
- `wait`: Maximum time to wait for the lock.
- `retry`: Delay between acquisition attempts.
- The lock is released either explicitly or at the end of the using block (even
  if the process crashes, the TTL ensures auto-release).

**Important Notes**:

- All endpoints must be independent Redis master nodes—not replicas of each
  other.
- Factory instances should be reused for performance.
- IsAcquired must always be checked.
- RedLock.net handles the correct release if only the client that holds the lock
  attempts to delete.

---

## Best Practices for Redis-Based Distributed Locks

Reliable distributed locking is a rigorously studied field, especially due to
the risk of **stale locks**, deadlocks, and split-brain scenarios. The following
best practices are advised irrespective of the precise language or Redis client
in use:

- **Always set a TTL (expiration) on the lock key.** Never leave locks
  open-ended.
- **Use a unique value for each process acquiring a lock.** This allows only the
  rightful owner to release its lock.
- **Release locks safely and atomically.** Release should validate ownership via
  Lua scripts.
- **For robust fault tolerance, deploy multiple, independent Redis nodes**, not
  master-slave or replicated topologies, for Redlock.
- **Monitor clock skew** and ensure slight variation does not result in errant
  lock revocations.
- **Avoid holding locks for extended periods.** Timeouts should be conservative.
- **Test for reliability under failure scenarios**: simulate crashes, high
  latency, or network partitions.
- **Monitor Redis health and latency**; degraded Redis performance may result in
  erroneous lock loss.
