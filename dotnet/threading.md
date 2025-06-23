---
title: Threading
---

## Thread Creation

```csharp
var t = new Thread(foo);
t.Start();
t.Join(); // Optionally wait for thread to finish execution

void foo() {
    Thread.Sleep(1000);
}
```

## Thread.Sleep(0) vs Thread.Yield()

- Sleep(0): The OS may reschedule the thread on a different CPU core
- Yield(): OS never migrates the thread to another core, preserving cache
  locality

## Local vs. Shared State

CLR assigns each thread its own memory stack, so that local variables are kept
separate. Objects captured through closures and static variables are shared.

## Foreground vs. Background Thread

_Foreground_ thread keeps process alive, even on main thread exit.
_Background_ thread gets closed on application exit.

## Signaling

Causes a thread to wait for a signal from another thread to continue its work.

```csharp
var signal = new ManualResetEvent(false);

/* --- thread 2 waiting for signal --- */
signal.WaitOne();
/* --- thread 2 waiting for signal --- */

signal.Set(); // notify the thread 2
```

## Threadpool

Reduces initial overhead of creating new threads, and context switching.

_Pooled threads_ are always _background_ threads.

## Task Creation

```csharp
var task = Task.Run (() =>{ Thread.Sleep (1000); });
task.Wait(); // blocks till completion
task.Result; // blocks till completion, and returns result
```

### Long-running Tasks

```csharp
// in case of long-running and blocking tasks:
Task task = Task.Factory.StartNew (() => { foo ();}, TaskCreationOptions.LongRunning);
```

### Continuations

```csharp
var awaiter = task.GetAwaiter();
awaiter.OnCompleted (() =>
{
 var result = awaiter.GetResult();
 // do something with result
});
```

### Waiting for multiple tasks

```csharp
await Task.WhenAll(task1, task2, ...);
```

### Async Streams

```csharp
async IAsyncEnumerable<int> foo(int count)
{
  for (var i = 0; i < count; i++)
  {
    await Task.Delay(1000);
    yield return i;
  }
}

await foreach (var number in foo(100))
{
  Console.WriteLine(number);
}
```

---

## Lazy initialization (thread safe)

### Lazy<T>

The `Lazy<T>` class is available to help with lazy initialization. If
instantiated with an argument of true, it implements the thread-safe
initialization pattern.

`Lazy<T>` actually implements a micro-optimized version of this pattern, called
double-checked locking. Double-checked locking performs an additional volatile
read to avoid the cost of obtaining a lock if the object is already initialized.

```csharp
Lazy<Expensive> _expensive = new Lazy<Expensive> (() => new Expensive(), true);

public Expensive Expensive { get { return _expensive.Value; } }
```

---

## Thread Local Storage

Each thread sees a separate copy of designated data.

### ThreadStatic

It doesn’t work with instance fields, nor does it play well with field
initializers.

```csharp
[ThreadStatic] static int local;
```

### ThreadLocal<T>

Provides thread-local storage for both static and instance fields, and allows
you to specify default values.

Factory function _lazy_ evaluates on the first call (for each thread).

```csharp
class Program
{
    static ThreadLocal<int> local = new ThreadLocal<int>(() => 5);

    static void Main()
    {
        var t1 = new Thread(() => Console.WriteLine($"Thread 1: {local.Value}"));
        t1.Start();
        t1.Join();
    }
}
```

### GetData and SetData

```csharp
class Test
{
    LocalDataStoreSlot slot = Thread.GetNamedDataSlot("Name");

    int SecurityLevel
    {
      get
      {
        object data = Thread.GetData (slot);
        return data == null ? 0 : (int) data; // null == uninitialized
      }
      set { Thread.SetData (slot, value); }
    }
}
```

### AsyncLocal<T>

When you update an `AsyncLocal<T>` value in an async method, that updated value
is visible to all the code running in that logical asynchronous flow—even if it
resumes on a different thread.

```csharp
class Program
{
    // Declare an AsyncLocal variable.
    static AsyncLocal<string> asyncLocalValue = new AsyncLocal<string>();

    static async Task Main()
    {
        // Set an initial value in the main async context.
        asyncLocalValue.Value = "Initial Value";
        Console.WriteLine($"Main starts with: {asyncLocalValue.Value}");

        // Call Task1, which updates the AsyncLocal variable.
        await Task1();

        // The value set by Task1 flows back to Main.
        Console.WriteLine($"Main after Task1: {asyncLocalValue.Value}");

        // Run Task2 in parallel to see separate logical flows.
        var task2 = Task.Run(() => Task2());
        await task2;

        // The main value remains unchanged by Task2 since it's a separate async flow.
        Console.WriteLine($"Main after Task2: {asyncLocalValue.Value}");
    }

    static async Task Task1()
    {
        // Update the AsyncLocal variable in this async flow.
        asyncLocalValue.Value = "Updated in Task1";
        Console.WriteLine($"Task1 start: {asyncLocalValue.Value}");

        // Simulate asynchronous work.
        await Task.Delay(100);
        Console.WriteLine($"Task1 end: {asyncLocalValue.Value}");
    }

    static async Task Task2()
    {
        // Set a different value in this separate async flow.
        asyncLocalValue.Value = "Updated in Task2";
        Console.WriteLine($"Task2 start: {asyncLocalValue.Value}");
        await Task.Delay(100);
        Console.WriteLine($"Task2 end: {asyncLocalValue.Value}");
    }
}
```

---

## Synchronization

- Exclusive locking: Allow just one thread to perform some activity or execute a
  section of code at a time. (lock, Mutex, SpinLock)

- Nonexclusive locking: Lets you limit concurrency. (Semaphore,
  ReaderWriterLock)

- Signaling: Allow a thread to block until receiving one or more notifications
  from other threads. (ManualResetEvent, AutoResetEvent, CountdownEvent,
  Barrier)

### lock

Simplifies thread synchronization by internally using _Monitor_. Protects
critical sections to ensure that only one thread executes them at a time.

- lock vs Monitor: Monitor offers `TryEnter`, `Wait`, `Pulse`, etc.—useful for
  timeouts or signaling
- Do not lock on `this`, publicly accessible objects, or value types.

```csharp
private readonly object _lock = new object();
lock (_lock)
{
  // Critical section
}
```

### Monitor

Provides more control than _lock_, allowing explicit `Enter` and `Exit` calls.

```csharp
private readonly object _monitor = new object();
Monitor.Enter(_monitor);
try
{
  // Critical section
}
finally
{
  Monitor.Exit(_monitor);
}
```

### SpinLock

Provides a low-level mutual exclusion lock that _avoids context switches_ by
spinning in place (for a short duration) while waiting for the lock. (_vs.
Mutex_)

- Busy-waits (spins) instead of context-switching
- Useful for low-latency, short lock durations
- Beware of CPU wastage and thread starvation

```csharp
private static SpinLock _spinLock = new SpinLock();
bool lockTaken = false;
try
{
  _spinLock.Enter(ref lockTaken);
  // Critical section
}
finally
{
  if (lockTaken)
    _spinLock.Exit();
}
```

### Mutex

Ensures exclusive access to a resource across multiple threads or _processes_.
By giving a Mutex a name like `"Global\\MyUniqueCrossProcessMutex"`, different
processes can reference the same underlying OS mutex, making it suitable for
scenarios where interprocess synchronization is required.

- Heavier due to kernel transitions; use only when inter-process needed

```csharp
private static Mutex _mutex = new Mutex();
_mutex.WaitOne();
try
{
  // Critical section
}
finally
{
  _mutex.ReleaseMutex();
}
```

### ReaderWriterLock/ReaderWriterLockSlim

Allows multiple threads to read concurrently while granting exclusive access for
writing.

- `Slim` is faster, supports recursion, avoids writer starvation
- Best used when read-heavy and write-light operations
- Read lock can be upgraded to write lock

```csharp
private static ReaderWriterLockSlim _rwLock = new ReaderWriterLockSlim();
_rwLock.EnterReadLock();
try
{
  // Read operation
}
finally
{
  _rwLock.ExitReadLock();
}
_rwLock.EnterWriteLock();
try
{
  // Write operation
}
finally
{
  _rwLock.ExitWriteLock();
}
```

```csharp
// Upgradeable Read Lock Sample
class Program
{
    static ReaderWriterLockSlim rwLock = new ReaderWriterLockSlim();
    static Dictionary<int, string> data = new Dictionary<int, string>();

    static void AddOrUpdate(int key, string value)
    {
        rwLock.EnterUpgradeableReadLock();
        try
        {
            if (!data.ContainsKey(key))
            {
                rwLock.EnterWriteLock();
                try
                {
                    data[key] = value;
                    Console.WriteLine($"Added: {key} = {value}");
                }
                finally
                {
                    rwLock.ExitWriteLock();
                }
            }
            else
            {
                Console.WriteLine($"Key {key} already exists with value: {data[key]}");
            }
        }
        finally
        {
            rwLock.ExitUpgradeableReadLock();
        }
    }

    static void Main()
    {
        AddOrUpdate(1, "A");
        AddOrUpdate(1, "B");
    }
}
```

### Semaphore/SemaphoreSlim

_Limits_ the number of threads that can access a resource concurrently.

- `SemaphoreSlim` is lighter and supports `async`/`await`
- Use Case: Pooling connections or limiting simultaneous tasks
- Pitfall: Releasing more times than acquired causes issues

```csharp
// Max 3 concurrent threads
private static SemaphoreSlim _semaphore = new SemaphoreSlim(3);
await _semaphore.WaitAsync();
try
{
  // Critical section
}
finally
{
  _semaphore.Release();
}
```

### AutoResetEvent

Notifies a waiting thread that an event has occurred; _automatically resets_
after releasing a single thread.

- Useful for producer–consumer patterns

```csharp
class AutoExample
{
    static AutoResetEvent autoEvt = new AutoResetEvent(false);

    static void Worker(int id)
    {
        Console.WriteLine($"[Auto] Worker {id} waiting");
        autoEvt.WaitOne();
        Console.WriteLine($"[Auto] Worker {id} released");
    }

    public static void Run()
    {
        for (int i = 1; i <= 3; i++)
            new Thread(() => Worker(i)).Start();

        Thread.Sleep(200);
        for (int i = 0; i < 3; i++)
        {
            autoEvt.Set(); // Releases exactly one waiting thread
            Thread.Sleep(100);
        }
    }
}
```

### ManualResetEvent

Notifies _one or more waiting threads_ that an event has occurred; remains
signaled until manually reset.

```csharp
class ManualExample
{
    static ManualResetEvent manualEvt = new ManualResetEvent(false);

    static void Worker(int id)
    {
        Console.WriteLine($"[Manual] Worker {id} waiting");
        manualEvt.WaitOne();
        Console.WriteLine($"[Manual] Worker {id} released");
    }

    public static void Run()
    {
        for (int i = 1; i <= 3; i++)
            new Thread(() => Worker(i)).Start();

        Thread.Sleep(200);
        manualEvt.Set(); // Releases ALL waiting threads
        // manualEvt.Reset(); // Optionally close the gate again
    }
}
```

### Barrier

Enables multiple threads to work concurrently until they all reach a certain
point (barrier), then they can proceed.

- Synchronizes multiple threads at checkpoints—great for phased workloads

```csharp
Barrier barrier = new Barrier(3);
// In each thread:
// Perform work
barrier.SignalAndWait();
// Continue after all threads reach the barrier
```

### CountdownEvent

Blocks until its count reaches zero, allowing threads to wait for multiple
operations to complete.

- Thread waits until count hits zero—common in fan-out/fan-in patterns

```csharp
CountdownEvent countdown = new CountdownEvent(3);
// In each thread:
// Perform work
countdown.Signal();
// In waiting thread:
countdown.Wait();
```

### Interlocked

Provides atomic operations for variables shared by multiple threads.

- Great for simple counters or flags

```csharp
private int _counter = 0;
Interlocked.Increment(ref _counter);
Interlocked.Decrement(ref _counter);
Interlocked.Add(ref _counter, 5);
```

---

## Parallel Programming

### Work distribution strategies

There are two strategies for partitioning work among threads: _data parallelism_
and _task parallelism_.

- Data Parallelism: Ideal for operations like processing arrays or collections
  where each element is handled independently (e.g., with `Parallel.For` or
  `PLINQ` in C#)

- Task Parallelism: Suitable for scenarios where different operations should be
  executed concurrently (e.g., in a web server handling separate client
  requests)

### PLINQ

`PLINQ` has the advantage of being easy to use in that it offloads the burden of
both work partitioning and result collation to .NET.

- Use `.AsParallel().AsOrdered()` to preserve result order.
- Use `AsUnordered()` when preserving result order is not necessary anymore.

```csharp
class Program
{
    static void Main()
    {
        // Create an array of numbers from 1 to 100.
        int[] numbers = Enumerable.Range(1, 100).ToArray();

        // Use PLINQ to filter out even numbers in parallel.
        var evenNumbers = numbers.AsParallel()
                                 .AsOrdered()
                                 .Where(n => n % 2 == 0)
                                 .ToArray();

        // Print the even numbers.
        Console.WriteLine("Even numbers:");
        foreach (int number in evenNumbers)
        {
            Console.Write(number + " ");
        }

        Console.WriteLine();
    }
}
```

### Parallel Class

#### Invoke

Executes multiple independent Actions in parallel.

```csharp
Parallel.Invoke(
    () => DoA(),
    () => DoB(),
    () => DoC()
);
```

#### For

Parallel equivalent of `for(int i = start; i < end; i++)`.

```csharp
Parallel.For(0, 10, i =>
{
    Console.WriteLine($"Index {i} on thread {Task.CurrentId}");
});
```

#### ForEach

Parallel version of `foreach(T item in collection)`.

```csharp
var data = new[] { "apple", "banana", "cherry" };
Parallel.ForEach(data, item =>
{
    Console.WriteLine($"Processing {item} on thread {Task.CurrentId}");
});
```

### Task Parallelism

#### TaskFactory

- Use `Task.Factory.StartNew(...)`for customizing task options, scheduler,
  state.
- Can create continuations: `ContinueWhenAll`, `ContinueWhenAny`.

```csharp
var task = Task.Factory.StartNew(state => Greet ("Hello"));
Console.WriteLine (task.AsyncState); // Greeting
task.Wait(); // Hello

void Greet (string message) { Console.Write (message); }
```

```csharp
Task.Factory.StartNew(
    () => DoCpuBoundWork(),
    CancellationToken.None,
    TaskCreationOptions.LongRunning, // Creates separate thread
    TaskScheduler.Default);
```

#### Task.Run

```csharp
var t = Task.Run(() => Compute());
await t;  // don't block, just await completion
```

#### Continuations & Coordination

```csharp
var a = Task.Run(() => A());
var b = Task.Run(() => B());
var both = await Task.WhenAll(a, b);
```

```csharp
Task task1 = Task.Factory.StartNew (() => Console.Write ("antecedent.."));
Task task2 = task1.ContinueWith (ant => Console.Write ("..continuation"));
```

#### Cancellation

```csharp
var cts = new CancellationTokenSource();
cts.CancelAfter(500);
var token = cts.Token;

var task = Task.Factory.StartNew (() =>
{
 Thread.Sleep (1000);
 token.ThrowIfCancellationRequested(); // Check for cancellation request
}, token);
```

#### ValueTask vs Task

- `Task<T>`:
  - always allocates a heap object
  - safe to await multiple times
  - best for general async methods

```csharp
public async Task<int> GetDataAsync(string key)
{
    await Task.Delay(50);  // I/O or computation
    return 42;
}
```

- `ValueTask<T>`:
  - avoids heap allocation when result completes synchronously
  - cannot be awaited or accessed multiple times unless converted using
    `.AsTask()`
  - ideal when there is high-frequency calls to an async method, synchronous
    completion is common, e.g. cached results

```csharp
private readonly Dictionary<string,int> _cache = new();

public ValueTask<int> GetCachedValueAsync(string key)
{
    if (_cache.TryGetValue(key, out var val))
        return new ValueTask<int>(val); // no heap allocation
    
    return new ValueTask<int>(FetchAndCacheAsync(key));
}
```

---

## Concurrent Collections

- `ConcurrentStack<T>`
- `ConcurrentQueue<T>`
- `ConcurrentBag<T>`: stores an unordered collection of objects (with duplicates
  permitted). It is suitable in situations for which you don’t care which
  element you get when calling `Take` or `TryTake`. The benefit of
  `ConcurrentBag<T>` over a concurrent queue or stack is that a bag's `Add`
  method suffers almost no contention when called by many threads at once
- `ConcurrentDictionary<TKey,TValue>`

The concurrent collections are optimized for high-concurrency scenarios; There
are some caveats, though:

- The conventional collections outperform the concurrent collections in all but
  highly concurrent scenarios
- A thread-safe collection doesn’t guarantee that the code using it will be
  thread safe
- If you enumerate over a concurrent collection while another thread is
  modifying it, no exception is thrown—instead, you get a mixture of old and new
  content.
- The concurrent stack, queue, and bag classes are implemented internally with
  linked lists. This makes them less memory-efficient than the nonconcurrent Stack
  and Queue classes, but better for concurrent access because linked lists are
  conducive to lock-free or low-lock implementations. (This is because inserting a
  node into a linked list requires updating just a couple of references, whereas
  inserting an element into a `List<T>` like structure might require moving
  thousands of existing elements.)

### `ConcurrentBag<T>`

Stores an unordered collection of objects (with duplicates permitted). It is
suitable in situations for which you don’t care which element you get when
calling `Take` or `TryTake`. The benefit of `ConcurrentBag<T>` over a concurrent
queue or stack is that a bag's `Add` method suffers almost no contention when
called by many threads at once. Inside a concurrent bag, each thread gets its
own private linked list. Elements are added to the private list that belongs to
the thread calling Add, eliminating contention.
