---
title: New Features
---

# A — Syntax improvements (with sample code)

These are mostly ergonomics — they reduce boilerplate and make intent clearer.

## 1. Primary constructors (C# 12)

Brief: define constructor parameters together with the type header; parameters
are in scope for the whole type body. Useful for thin data/DI holders.

```csharp
// C# 12 primary constructor (class)
public class Person(string firstName, string lastName)
{
    // parameters are in scope; backing fields are generated
    public string FullName => firstName + " " + lastName;
}

// usage
var p = new Person("Ada", "Lovelace");
Console.WriteLine(p.FullName); // Ada Lovelace
```

Notes: adding a primary constructor suppresses an implicit parameterless
constructor; explicit constructors must call it with `this(...)`.

---

## 2. Collection expressions (C# 12)

Brief: `[...]` collection expressions and spread (`..`) let you write collection
literals and combine them more concisely. Compiler binds them to types that
implement expected collection patterns.

```csharp
// collection expressions (C# 12)
List<int> more = [1, 2, 3];
var all = [0, ..more, 4]; // spread "more" into new collection expression

void Print(IEnumerable<int> seq) {
    foreach (var i in seq) Console.WriteLine(i);
}
Print([10, 20, 30]);
```

Notes: exact shape/binding rules are in the proposal; not every collection
expression allocates the same way — depends on compiler/BCL helpers.

---

## 3. Inline arrays (C# 12)

Brief: declare small fixed-size arrays inside `struct`s (no separate heap
array). Helpful for interop and very low-allocation scenarios.

```csharp
// conceptual example — inline arrays are declared on struct layout
public struct SmallVector
{
    public int length;
    public int inline[4]; // pseudo-syntax for illustration (layout-supporting)
}

// usage: SmallVector can contain 4 ints inline without extra heap allocation
```

Note: exact syntax and usage require the C# compiler and runtime support; this
feature reduces heap allocations for small fixed buffers.

---

## 4. Optional parameters in lambdas (C# 12)

Brief: lambdas may declare default parameter values.

```csharp
Func<int,int> add = (int x = 5) => x + 1;
Console.WriteLine(add()); // 6
```

---

## 5. `params` collections (C# 13)

Brief: `params` works with collection expressions and more flexible call-site
shapes. Useful for varargs ergonomics with collection expressions.

```csharp
void M(params int[] values) { Console.WriteLine(values.Length); }
M([1,2,3]); // works with collection expressions
```

---

## 6. Implicit indexer access in object initializers (C# 13)

Brief: set indexers during initialization with a terser syntax.

```csharp
var dict = new Dictionary<string,int> { ["one"] = 1, ["two"] = 2 };
```

(This style was earlier, but C# 13 improved binding in certain initializer
contexts.)

---

## 7. New escape sequence `\e` (C# 13)

Brief: shorthand escape for ESC (ASCII 27).

```csharp
Console.Write("\e"); // prints ESC control character
```

---

## 8. `nameof` unbound generic types (C# 14)

Brief: `nameof(List<>)` is supported. Small ergonomics for reflection/codegen.

```csharp
Console.WriteLine(nameof(List<>)); // "List`1" depending on display
```

---

## 9. Lambda parameter modifiers + field-backed property access (C# 14)

Brief: allow `ref`/`in`/`out` modifiers on simple lambda parameters without
repeating types; `field` contextual keyword inside properties to reference
backing field.

```csharp
// lambda with modifier (C# 14)
Action refAction = ref (ref int x) => x = 42; // simplified illustration

// field-backed property
public int Value
{
    get => field;   // 'field' refers to automatically generated backing field
    init => field = value;
}
```

---

# B — New / semantic features (binding, behavior, or capability changes)

These may affect emitted code, overload resolution, or allow patterns previously
impossible.

- **Primary constructors (semantic aspects)**: parameter scope and
  default-constructor suppression change binding/initialization semantics.
- **Ref/unsafe in iterators & async methods, ref struct interfaces (C# 13)**:
  expands where `ref` semantics can appear (more low-level patterns in
  async/iterators). These are not just sugar — they change what you can express
  and how values are passed.
- **New lock object & lock semantics (C# 13)**: language-level changes that
  alter lock patterns and correctness — has concurrency semantics implications.
- **Partial properties / indexers (C# 13)**: improves source-gen & partial-type
  workflows (generator + partial class interactions). Good for MVVM source
  generators and other tooling.
- **Extension-members expansion (C# 14)**: broader extension capabilities
  (affects binding and how code can be extended without altering types).

---

# C — Compile-time, runtime, JIT & Native-AOT performance impacts

Here I separate _purely syntactic_ items (no runtime effect) from changes that
do affect performance or codegen.

> **Rule of thumb:** If the feature only rewrites AST/IL without changing layout
> or runtime helpers, its direct runtime cost is zero (though ergonomics can
> lead to better micro-optimizations). If it changes _type layout_,
> _marshalling_, or _binding to new BCL helpers_, then runtime / JIT /
> native-AOT can be affected.

## Features with meaningful runtime / allocation / layout impact

### 1) Inline arrays (C# 12) — **memory & locality improvements**

- What: small fixed-size arrays can live inside a `struct` instead of holding a
  reference to a heap array.
- When it helps: hot loops, parsers, codecs, small buffers passed around
  frequently — removes small heap allocations and improves cache locality.
- What to watch: layout and marshaling semantics change; if you need Native AOT
  or interop, ensure the target runtime/SDK supports the same layout and that
  any marshalling attributes are correct.
- Interview point: inline arrays reduce GC pressure and pointer-chasing for very
  small buffers — a real win in micro-benchmarks and high-throughput code.

### 2) `Span<T>` and first-class span/implicit conversions (C# 14 and later ecosystem improvements) — **zero-allocation memory API improvements**

- What: language & BCL work to make `Span<T>` / `ReadOnlySpan<T>` easier to use
  (less overload explosion, improved conversions). These are `ref struct`
  improvements that enable zero-allocation patterns more ergonomically.
- When it helps: I/O, parsing, slicing buffers, codecs, and any API that wants
  to avoid allocations.
- JIT impact: `ref struct` usage is stack-bound; the JIT will often inline and
  specialize methods operating on spans — fewer heap allocations, fewer GC
  pauses.
- Native AOT: `Span<T>` is friendly to AOT as long as the BCL usages are
  available; still check concrete AOT runtime support.

### 3) Collection expressions & `params` collection improvements — **potential compile-time optimizations**

- What: collection expressions allow the compiler and BCL to collaborate to
  allocate fewer temporaries in some patterns. The benefit is situational —
  depends on whether the compiler emits calls to specialized factory methods
  that avoid intermediate allocations.
- Interview point: syntactic sugar can surface optimization opportunities; but
  don’t assume zero allocations until you inspect the emitted IL or benchmark.

---

## Compile-time & JIT impacts (general)

- **Better type inference (method-group natural type)** reduces the need for
  wrapper delegates or hand-written adapters — this can reduce tiny allocations
  and enable cleaner JIT inlining patterns.
- **Primary constructors**: small compile-time simplification; emitted IL for
  constructors may be slightly simpler, but runtime performance change is minor.

---

## Native AOT / Native builds considerations

- Most language features are _compiler_ features only and are compatible with
  Native AOT. But features that change **type layout** (inline arrays) or rely
  on new **BCL runtime helpers** (span convenience APIs, collection expression
  helpers) require the runtime/BCL to support them under Native AOT. Always test
  the exact SDK/runtime you target. Microsoft’s .NET 10 notes and C# release
  docs emphasize verifying runtime support for layout or marshalling changes.

---

# Quick cheat sheet for interviews

- **Syntactic only (no runtime change):** collection expressions (often),
  primary constructors (mostly), optional lambda params, `\e`.
- **Semantic / runtime-affecting:** inline arrays (layout & allocation),
  Span-related improvements (zero-alloc APIs), ref/unsafe support in
  async/iterators (changes what is expressible).
- **Native-AOT note:** language support ≠ runtime/AOT support. For inline arrays
  / spans, verify target runtime.
