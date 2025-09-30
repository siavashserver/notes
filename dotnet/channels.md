---
title: Channels
---

## C# Channels

Channels are a modern, low-level building block for async producer/consumer
pipelines that avoid locks and blocking calls while supporting backpressure,
bounded capacity, and high-throughput single-producer/single-consumer scenarios.
They sit between in-memory concurrent collections and full message brokers,
designed for async/await workflows and scalable background processing.

---

### Key concepts and when to use them

- Channel<T>: an async-safe queue with separated Writer and Reader endpoints;
  supports bounded and unbounded modes.
- Bounded vs Unbounded: bounded channels provide backpressure (WriteAsync delays
  or fails when full) and are preferred when you must limit memory/throughput;
  unbounded is simpler but risks memory growth.
- SingleWriter/SingleReader options: micro-optimizations that remove
  synchronization overhead when usage is known ahead of time.
- ReadAllAsync / WaitToReadAsync: ergonomic, allocation-friendly reader patterns
  for consuming until completion.
- Complete/CompleteAsync + TryComplete: graceful shutdown and exception
  propagation from writers to readers.  
  Use channels when you need reliable in-process streaming between producers and
  consumers, want async flow-control (backpressure), or need a composition
  primitive for pipelines and long-running background services.

---

### Minimal producer/consumer (unbounded) — simple example

```csharp
using System;
using System.Threading;
using System.Threading.Channels;
using System.Threading.Tasks;

var cts = new CancellationTokenSource();
var channel = Channel.CreateUnbounded<int>();

// Producer
Task producer = Task.Run(async () =>
{
    for (int i = 0; i < 20; i++)
    {
        await channel.Writer.WriteAsync(i, cts.Token);
        Console.WriteLine($"Produced {i}");
        await Task.Delay(50, cts.Token);
    }
    channel.Writer.Complete();
});

// Consumer
Task consumer = Task.Run(async () =>
{
    await foreach (var item in channel.Reader.ReadAllAsync(cts.Token))
    {
        Console.WriteLine($"Consumed {item}");
        await Task.Delay(120, cts.Token);
    }
});

await Task.WhenAll(producer, consumer);
```

This shows basic async production and consumption with ReadAllAsync, which
completes after Writer calls Complete.

---

### Bounded channel with backpressure and graceful shutdown

```csharp
var options = new BoundedChannelOptions(capacity: 5)
{
    FullMode = BoundedChannelFullMode.Wait, // backpressure: writers await when full
    SingleReader = false,
    SingleWriter = false
};
var channel = Channel.CreateBounded<string>(options);
var cts = new CancellationTokenSource();

// Producers (multiple)
var producers = Enumerable.Range(1, 3).Select(p => Task.Run(async () =>
{
    for (int i = 0; i < 10; i++)
    {
        var item = $"P{p}-{i}";
        await channel.Writer.WriteAsync(item, cts.Token); // may await if channel is full
        Console.WriteLine($"Produced {item}");
    }
})).ToArray();

// Consumer (single)
var consumer = Task.Run(async () =>
{
    try
    {
        await foreach (var item in channel.Reader.ReadAllAsync(cts.Token))
        {
            Console.WriteLine($"Consumed {item}");
            await Task.Delay(200, cts.Token);
        }
    }
    catch (OperationCanceledException) { }
});

// When all producers finish, signal completion
await Task.WhenAll(producers);
channel.Writer.Complete();
await consumer;
```

Bounded channels give deterministic behavior under load; writers will await
rather than causing unbounded memory growth.

---

### Error propagation and partial-failure handling

- If a writer hits an exception and you call Writer.Complete(ex), readers will
  observe that exception when enumerating ReadAllAsync or calling
  TryRead/TryPeek, enabling centralized error handling.
- Use TryWrite for fire-and-forget attempts; check result to avoid deadlocks
  when channel is full.

Example: writer signals an error:

```csharp
channel.Writer.TryComplete(new InvalidOperationException("upstream failed"));
```

Readers enumerating ReadAllAsync will rethrow that exception when they reach the
end of the stream.

---

### High-performance patterns and tips

- Prefer SingleWriter/SingleReader flags when applicable for lower contention.
- Use BoundedChannelFullMode.DropWrite or DropOldest for lossy streams (e.g.,
  metrics telemetry) where throughput matters more than strict delivery.
- Avoid mixing blocking waits (Wait/Result) with async channel APIs; use async
  all the way to benefit from the cooperative scheduling model.
- Pool objects that cross the channel to reduce GC pressure (ArrayPool,
  ObjectPool) for high-throughput scenarios.
- Measure: channels are in-memory constructs; for cross-process or cross-machine
  scenarios use real message brokers.

---

### Composition examples (fan-out, pipeline)

- Fan-out: one producer, multiple consumers reading with TryRead/ReadAllAsync
  concurrently to parallelize processing.
- Pipeline: chain Channel<T> between pipeline stages where each stage processes
  items and writes to the next channel, ensuring per-stage concurrency and
  backpressure.

Pipeline skeleton:

```csharp
var stage1 = Channel.CreateBounded<Item>(100);
var stage2 = Channel.CreateBounded<Processed>(100);

// Stage 1 worker(s) read source -> write to stage1.Writer
// Stage 2 worker(s) await stage1.Reader.ReadAllAsync and write to stage2.Writer
// Final consumer drains stage2.Reader.ReadAllAsync
```

---

### How channels compare to other constructs

- vs ConcurrentQueue/BlockingCollection: channels are async-first and avoid
  blocking threads; BlockingCollection is synchronous and blocks threads on
  Wait; channels integrate with async/await and CancellationToken naturally.
- vs Dataflow (TPL Dataflow): Dataflow offers richer transforms and built-in
  blocks (TransformBlock, ActionBlock) but is a heavier dependency; channels
  provide a lighter, lower-level primitive you can compose yourself.
- vs Message brokers: channels are in-process and extremely fast; they do not
  provide durability, cross-process delivery, or complex routing.

---

### Useful APIs to remember

- Channel.CreateUnbounded<T>(), Channel.CreateBounded<T>(options)
- channel.Writer.WriteAsync / TryWrite / TryComplete / Complete
- channel.Reader.ReadAsync / TryRead / ReadAllAsync / WaitToReadAsync
- BoundedChannelOptions and BoundedChannelFullMode for backpressure strategies.
