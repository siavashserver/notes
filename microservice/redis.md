---
title: Redis
---

It's an in-memory data‑structure store used for cache, session, messaging, etc., offering low-latency reads/writes.

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
