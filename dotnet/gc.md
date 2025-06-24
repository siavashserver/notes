---
title: Garbage Collector
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
