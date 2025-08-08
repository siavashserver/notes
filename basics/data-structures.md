---
title: Data Structures
---

## Big-O Notation: Why it matters

**Big-O notation** characterizes how the cost (time or space) of operations
grows with input size.

| Data Structure                      | Access   | Search                   | Insert               | Delete               | Notes                                                     |
| ----------------------------------- | -------- | ------------------------ | -------------------- | -------------------- | --------------------------------------------------------- |
| **Array / List<T>** (dynamic array) | O(1)     | O(n)                     | Amortized O(1)       | O(n)                 | Resize cost amortized; good for index access.             |
| **Linked List** (singly/doubly)     | O(n)     | O(n)                     | O(1) (if node known) | O(1) (if node known) | No random access; cheap inserts/removals given node.      |
| **Stack** (LIFO)                    | —        | O(n)                     | O(1)                 | O(1)                 | Usually implemented on array or linked list.              |
| **Queue** (FIFO)                    | —        | O(n)                     | O(1)                 | O(1)                 | Circular buffer or linked list.                           |
| **Hash Table / Dictionary**         | —        | O(1) average, O(n) worst | O(1) avg             | O(1) avg             | Collision handling and load factor matter.                |
| **Binary Search Tree (balanced)**   | O(log n) | O(log n)                 | O(log n)             | O(log n)             | Sorted order; e.g., SortedDictionary uses red-black tree. |
| **Heap (priority queue)**           | —        | O(n)                     | O(log n)             | O(log n)             | Good for kth element, scheduling.                         |

To calculate the **Big-O of a function**, you're analyzing how the **runtime or
memory usage grows** relative to input size (**n**) as **n → ∞**. You focus on
the **dominant term** and ignore constants and lower-order terms.

**By default, Big-O notation refers to the dominant operation in the data
structure’s typical use case — most commonly:**

- **Access time** for arrays or lists
- **Search time** for trees, hash tables, sets, etc.
- **Insert/delete time** for stacks, queues, and dynamic data structures
- **Traversal time** for graphs

### How to calculate Big-O of a given function

1. **Identify input size (n)**
2. **Break down loops and recursive calls**
3. **Focus on dominant growth**
4. **State Big-O with explanation, not just a number**

| Code Pattern                                         | Big-O    |
| ---------------------------------------------------- | -------- |
| Single loop over n items                             | O(n)     |
| Nested loop over n                                   | O(n²)    |
| Loop with step doubling (i \*= 2)                    | O(log n) |
| Two separate loops                                   | O(n)     |
| Recursion dividing n by 2                            | O(log n) |
| Recursion combining all subresults (e.g., Fibonacci) | O(2ⁿ)    |
| Loop from 1 to √n                                    | O(√n)    |
| Loop from 1 to log n                                 | O(log n) |

#### Example 1: Simple loop

```csharp
void PrintAll(int[] arr) {
    foreach (var item in arr)
        Console.WriteLine(item);
}
```

- Loop runs **n** times → **O(n)**

#### Example 2: Nested loop

```csharp
void PrintPairs(int[] arr) {
    for (int i = 0; i < arr.Length; i++)
        for (int j = 0; j < arr.Length; j++)
            Console.WriteLine($"{arr[i]}, {arr[j]}");
}
```

- Outer: n, Inner: n → **O(n²)**

#### Example 3: Two separate loops

```csharp
void PrintTwice(int[] arr) {
    foreach (var a in arr) Console.WriteLine(a);
    foreach (var b in arr) Console.WriteLine(b);
}
```

- O(n) + O(n) → **O(n)** (drop constant factor)

#### Example 4: Logarithmic loop

```csharp
void Halve(int n) {
    while (n > 1) {
        Console.WriteLine(n);
        n = n / 2;
    }
}
```

- n → n/2 → n/4 → ... → 1 → **log₂(n) steps** → **O(log n)**

#### Example 5: Recursive function

```csharp
int Fibonacci(int n) {
    if (n <= 1) return n;
    return Fibonacci(n - 1) + Fibonacci(n - 2);
}
```

- Recursion tree is exponential → **O(2ⁿ)**

#### Example 6: Binary search

```csharp
int BinarySearch(int[] arr, int x) {
    int low = 0, high = arr.Length - 1;
    while (low <= high) {
        int mid = (low + high) / 2;
        if (arr[mid] == x) return mid;
        else if (x < arr[mid]) high = mid - 1;
        else low = mid + 1;
    }
    return -1;
}
```

- Each step cuts search space in half → **O(log n)**

---
