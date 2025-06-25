---
title: Garbage Collector
---

## Generational Garbage Collection

.NET GC uses a _generational_, _tracing_ (**not reference counting**) collector
optimizing for the fact that most objects die young. When the .NET GC performs a
_mark (tracing) collection_, it starts by identifying _root objects_.

_Roots_ are references that the GC always considers _alive_ and from which it
explores the object graph:

- _Local variables_ and parameters on thread stacks
- _CPU registers_ that reference heap objects
- _Static fields_ and thread-statics
- _GC handles_ (e.g. pinned, strong, weak handles)
- _Finalization queue entries_ for non-finalized objects

Once roots are identified:

- The GC marks each referenced object as alive
- It then recursively visits all object fields using type metadata to find
  pointers, marking each visited object
- The process continues until all reachable objects have been marked

After marking:

- _Sweep & compact_ the Small Object Heap (SOH), relocating live objects and
  updating references
- _LOH and POH_ swept but not compacted (unless compaction is explicitly
  enabled)
- All unmarked objects are reclaimed as garbage

There are three generations:

- Gen 0: brand-new, short-lived objects
- Gen 1: objects that survived one _Gen 0_ collection
- Gen 2: long-living objects; also includes _LOH_, logically treated as _Gen 3_
  but physically in _Gen 2_ collection

Whenever a _GC_ happens, surviving objects get moved to a higher generation. A
_Full GC_, including all generations and _LOH_, happens when _Gen 2_ gets full
as well.

`GC.Collect()` forces GC to run.

## GC Phases

- Marking: identify live objects via roots (stacks, statics, handles)
- Relocating: update references for moving objects
- Compacting: move live objects (on SOH) to the end of the heap

## GC Modes

- Workstation GC: optimized for desktop responsiveness; can be concurrent
- Server GC: optimized for throughput/server; multiple heaps per logical CPU,
  parallel collections

## Heaps: SOH vs LOH

- SOH (Small Object Heap) stores objects smaller than 85KB across _Gen 0,1,2_
- LOH (Large Object Heap) contains objects bigger than 85KB, including _large
  arrays_ and _static objects_
  - LOH objects are only collected during _Full GC_
  - No automatic compaction by default

## Pinned Object Heap (POH)

POH holds objects that are pinned, so they aren't moved during GC, improving
performance in interop and reducing fragmentation on LOH. It runs alongside
Gen 2/LOH.

Use POH when you need long-lived pinned buffers, especially for interop with
native code (e.g., image processing, network, file I/O). Prefer short-lived
`fixed` scope pinning for quick native interop, leaving long-lived buffers to
POH.

```csharp
var buffer = GC.AllocateArray<byte>(1024, pinned: true);
```

## Finalizer Thread

.NET creates a single dedicated finalizer thread per process whose sole job is
to run `Finalize()` methods on objects queued for cleanup. _Blocking or slow_
finalizers stall this thread, delaying cleanup and causing memory pressure or
OOM.

## Strong vs. Weak Reference

- A strong reference is the normal reference you assign in C#. As long as a
  strong reference exists, the GC will not collect the object.

```csharp
var obj = new MyClass();  // strong reference to MyClass instance
// `obj` keeps the object alive; GC won't collect it.
```

- A weak reference lets you reference an object without preventing it from being
  collected. If no strong references exist, the GC may reclaim the object, even
  if a weak reference points to it. Use cases:
  - _Cache scenarios_: store objects only while memory allows
  - _Event listeners_: avoid memory leaks by not holding subscribers alive

```csharp
// Create a weak reference to a new object
WeakReference<MyClass> weakRef = new WeakReference<MyClass>(new MyClass());

// Remove strong references
// ...

if (weakRef.TryGetTarget(out MyClass target))
{
    // Object is still alive
    target.DoWork();
}
else
{
    // It was collected
    Console.WriteLine("Object has been garbage-collected");
}
```

### Short vs. Long Weak References

- _Short weak references_ (default): object becomes eligible for collection even
  if it has a finalizer
- _Long weak references_ (pass `true` in constructor): keep the object alive
  through finalization, reclaimed later, but state may be unpredictable

```csharp
WeakReference wr = new WeakReference(obj, trackResurrection: true);
```

---

## `Span<T>` and `Memory<T>`

They help with writing low-allocation code that minimizes managed memory
allocations, and also enable slicing working with a portion of an array, string,
or memory block without creating a copy.

`Span<T>` can wrap stack-allocated memory, `Memory<T>` cannot wrap
stack-allocated memory.

### Span

`Span<T>` and `ReadOnlySpan<T>` are defined as ref structs to maximize their
optimization potential as well as allowing them to work safely with
stack-allocated memory (as you'll see in the next section). However, it also
imposes limitations. In addition to being array-unfriendly, you cannot use them
as fields in a class (this would put them on the heap). This, in turn, prevents
them from appearing in lambda expressions and as parameters in asynchronous
methods, iterators, and asynchronous streams.

```csharp
// Sum the middle 500 elements (starting from position 250):
int total = Sum (numbers.AsSpan (250, 500));

Span<int> span = numbers;
Console.WriteLine(span [^1]); // Last element
Console.WriteLine(Sum(span[..10])); // First 10 elements
Console.WriteLine(Sum(span[100..])); // 100th element to end
Console.WriteLine(Sum(span[^5..])); // Last 5 elements

int Sum(ReadOnlySpan<int> numbers)
{
 int total = 0;
 foreach (int i in numbers) total += i;
 return total;
}
```

### Memory

The `Memory<T>` and `ReadOnlyMemory<T>` structs act as spans that cannot wrap
stack-allocated memory, allowing their use in fields, lambda expressions,
asynchronous methods, and so on.

```csharp
Memory<int> mem1 = new int[] { 1, 2, 3 };
var mem2 = new int[] { 1, 2, 3 }.AsMemory();
```
