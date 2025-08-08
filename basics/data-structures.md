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

## Data Structures

### Big-O Complexity Cheat-Sheet

| Data Structure             | Access |  Search  | Insertion | Deletion |
| -------------------------- | :----: | :------: | :-------: | :------: |
| Array                      |  O(1)  |   O(n)   |   O(n)    |   O(n)   |
| Singly Linked List         |  O(n)  |   O(n)   |  O(1)\*   |   O(n)   |
| Doubly Linked List         |  O(n)  |   O(n)   |  O(1)\*   |  O(1)\*  |
| Stack                      |  O(n)  |   O(n)   | **O(1)**  | **O(1)** |
| Queue                      |  O(n)  |   O(n)   | **O(1)**  | **O(1)** |
| Deque                      |  O(n)  |   O(n)   | **O(1)**  | **O(1)** |
| Dictionary / HashMap       |   –    |  O(1)†   |   O(1)†   |  O(1)†   |
| Unordered Binary Tree      |   –    |   O(n)   |  O(1)\*   |  O(?)\*  |
| Binary Search Tree (avg)   |   –    | O(log n) | O(log n)  | O(log n) |
| Binary Search Tree (worst) |   –    |   O(n)   |   O(n)    |   O(n)   |
| AVL / Self-Balancing Tree  |   –    | O(log n) | O(log n)  | O(log n) |
| Red-Black Tree             |   –    | O(log n) | O(log n)  | O(log n) |

- \* Given a pointer/reference to the node
- † Average case; collisions may degrade to O(n)

### Array

- Fixed-size, contiguous block of memory.
- O(1) random access, but resizing/inserting/deleting in middle is O(n).

```csharp
// Fixed-size array of T
T[] arr = new T[capacity];
// Access:
T x = arr[i];         // O(1)
// Update:
arr[i] = newValue;    // O(1)
```

### Singly Linked List

- Nodes hold `Value` + `Next` pointer.
- Cheap inserts/removals at head (O(1)), but O(n) search/traverse.

```csharp
class ListNode<T>
{
    public T Value;
    public ListNode<T> Next;
    public ListNode(T val) => Value = val;
}

class SinglyLinkedList<T>
{
    private ListNode<T> head;
    public void AddFirst(T val)
    {
        var node = new ListNode<T>(val) { Next = head };
        head = node;
    }
    public bool Remove(T val)
    {
        ListNode<T> prev = null, curr = head;
        while (curr != null)
        {
            if (EqualityComparer<T>.Default.Equals(curr.Value, val))
            {
                if (prev == null) head = curr.Next;
                else prev.Next = curr.Next;
                return true;
            }
            prev = curr;
            curr = curr.Next;
        }
        return false;
    }
    public bool Contains(T val)
    {
        var curr = head;
        while (curr != null)
        {
            if (EqualityComparer<T>.Default.Equals(curr.Value, val)) return true;
            curr = curr.Next;
        }
        return false;
    }
}
```

### Doubly Linked List

- Each node has `Prev` and `Next`.
- Bi-directional traversal; O(1) insert/remove given node reference.

```csharp
class DListNode<T>
{
    public T Value;
    public DListNode<T> Prev, Next;
    public DListNode(T val) => Value = val;
}

class DoublyLinkedList<T>
{
    private DListNode<T> head, tail;
    public void AddFirst(T val)
    {
        var node = new DListNode<T>(val);
        if (head == null) head = tail = node;
        else
        {
            node.Next = head;
            head.Prev = node;
            head = node;
        }
    }
    public void AddLast(T val)
    {
        var node = new DListNode<T>(val);
        if (tail == null) head = tail = node;
        else
        {
            tail.Next = node;
            node.Prev = tail;
            tail = node;
        }
    }
    public bool Remove(T val)
    {
        var curr = head;
        while (curr != null)
        {
            if (EqualityComparer<T>.Default.Equals(curr.Value, val))
            {
                if (curr.Prev != null) curr.Prev.Next = curr.Next;
                else head = curr.Next;
                if (curr.Next != null) curr.Next.Prev = curr.Prev;
                else tail = curr.Prev;
                return true;
            }
            curr = curr.Next;
        }
        return false;
    }
}
```

### Stack (LIFO)

- Push/pop only at top.
- Often backed by array or linked list; all ops O(1).

```csharp
class Stack<T>
{
    private List<T> data = new List<T>();
    public void Push(T val) => data.Add(val);           // O(1) amortized
    public T Pop()
    {
        if (data.Count == 0) throw new InvalidOperationException();
        var val = data[^1];
        data.RemoveAt(data.Count - 1);                  // O(1)
        return val;
    }
    public T Peek() => data.Count > 0 ? data[^1] : throw new InvalidOperationException();
}
```

### Queue (FIFO)

- Enqueue at tail, dequeue at head.
- O(1) if using a linked list or circular buffer.

```csharp
class Queue<T>
{
    private LinkedList<T> list = new LinkedList<T>();
    public void Enqueue(T val) => list.AddLast(val);   // O(1)
    public T Dequeue()
    {
        if (list.Count == 0) throw new InvalidOperationException();
        var val = list.First.Value;
        list.RemoveFirst();                             // O(1)
        return val;
    }
    public T Peek() => list.First != null ? list.First.Value : throw new InvalidOperationException();
}
```

### Deque (Double-ended Queue)

- Insert/delete at both ends in O(1).
- Can be built atop a doubly linked list or circular buffer.

```csharp
class Deque<T>
{
    private DoublyLinkedList<T> list = new DoublyLinkedList<T>();
    public void AddFront(T val) => list.AddFirst(val);  // O(1)
    public void AddBack(T val)  => list.AddLast(val);   // O(1)
    public T RemoveFront() => // similar to Queue.Dequeue()
        throw new NotImplementedException();
    public T RemoveBack()  => // mirror of RemoveFront
        throw new NotImplementedException();
}
```

_(Implement removes analogously to `DoublyLinkedList.RemoveFirst/RemoveLast`.)_

### Dictionary / HashMap

- Key → value store via hashing.
- Average-case O(1) insert/search/delete; worst-case O(n) if many collisions.
- In .NET use `Dictionary<K,V>`.

```csharp
var dict = new Dictionary<string, int>();
dict["apple"] = 5;                     // O(1)
bool has = dict.ContainsKey("apple");  // O(1)
int value = dict["apple"];             // O(1)
dict.Remove("apple");                  // O(1)
```

### Binary Tree (Unordered)

- Each node has up to two children.
- No ordering invariant; typically used for generic hierarchical data.

```csharp
class TreeNode<T>
{
    public T Value;
    public TreeNode<T> Left, Right;
    public TreeNode(T val) => Value = val;
}
```

### Binary Search Tree (BST)

- Left subtree < node < right subtree.
- Average O(log n) search/insert/delete; worst O(n) if unbalanced.

```csharp
class BST<T> where T : IComparable<T>
{
    private TreeNode<T> root;
    public void Insert(T val) => root = InsertRec(root, val);
    private TreeNode<T> InsertRec(TreeNode<T> node, T val)
    {
        if (node == null) return new TreeNode<T>(val);
        int cmp = val.CompareTo(node.Value);
        if (cmp < 0) node.Left  = InsertRec(node.Left,  val);
        else if (cmp > 0) node.Right = InsertRec(node.Right, val);
        return node;
    }
    public bool Search(T val) => SearchRec(root, val) != null;
    private TreeNode<T> SearchRec(TreeNode<T> node, T val)
    {
        if (node == null) return null;
        int cmp = val.CompareTo(node.Value);
        if (cmp == 0) return node;
        return (cmp < 0) ? SearchRec(node.Left, val) : SearchRec(node.Right, val);
    }
    // Deletion omitted for brevity…
}
```

### Self-Balancing Search Tree (e.g. AVL)

- Maintains height‐balance to ensure O(log n) worst-case.
- AVL uses node‐height and rotations on imbalance.

```csharp
class AvlNode<T>
{
    public T Value;
    public AvlNode<T> Left, Right;
    public int Height = 1;
    public AvlNode(T val) => Value = val;
}

class AvlTree<T> where T : IComparable<T>
{
    private AvlNode<T> root;
    public void Insert(T val) => root = InsertRec(root, val);
    private AvlNode<T> InsertRec(AvlNode<T> node, T val)
    {
        if (node == null) return new AvlNode<T>(val);
        int cmp = val.CompareTo(node.Value);
        if      (cmp < 0) node.Left  = InsertRec(node.Left,  val);
        else if (cmp > 0) node.Right = InsertRec(node.Right, val);
        else return node; // no duplicates
        UpdateHeight(node);
        return Balance(node);
    }
    // ... UpdateHeight, Balance, RotateLeft/Right implementations omitted for brevity ...
}
```

### Red-Black Tree

- Another O(log n) worst-case balanced BST.
- Each node colored red/black with invariants ensuring balanced paths.
- .NET offers `SortedDictionary<K,V>` (RB-tree underneath).

```csharp
// In .NET you can simply use:
var rb = new SortedDictionary<int, string>();
rb[10] = "ten";   // O(log n)
string s = rb[10]; // O(log n)
```
