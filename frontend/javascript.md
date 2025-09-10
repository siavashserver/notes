---
title: JavaScript
---

## JavaScript & ECMAScript

- **JavaScript** is the widely used scripting language, whose core syntax and
  behavior are defined by the **ECMAScript** standard (ECMA‑262).
- ECMAScript evolves through annual edition releases and proposal stages
  (Stage 0 to Stage 4) under the **TC39 process**.
- Key milestones include:

  - **ES6/ES2015**: introduced `let`, `const`, arrow functions, modules,
    classes, promises.
  - **Subsequent editions**: ES2017 (`async`/`await`), ES2018 (spread/rest,
    `Promise.finally`), ES2019 (`Array.flat`, `Object.fromEntries`).
  - **ES2024**: newer additions like `Object.groupBy()` and `Map.groupBy()` for
    more expressive data manipulation.

- The **specification defines only the language core**, excluding runtime
  constructs like I/O or DOM; these are provided by **host environments**.

## The V8 JavaScript Engine

- **V8** is Google’s open-source, high-performance **JavaScript and WebAssembly
  engine**, written in C++, used in both Chrome and Node.js
- It implements ECMAScript by **compiling JavaScript directly into highly
  optimized machine code**, leveraging JIT compilation, inline caching, hidden
  classes, and efficient garbage collection.
- **Internals**: code is parsed into an AST, interpreted via Ignition to
  bytecode, then optimized by the Turbofan compiler; V8 deoptimizes if
  assumptions break at runtime.
- V8 can be embedded in C++ applications and runs across multiple architectures
  (x86, ARM, MIPS, PPC, etc.).

## Node.js: JavaScript at the Server

- **Node.js** is an open-source, cross-platform **JavaScript runtime** built on
  V8, enabling JavaScript execution outside browsers.
- It provides server-side capabilities such as filesystem access, networking,
  cryptography, streams, and process management—none of which are defined in
  ECMAScript.
- Internally, Node.js leverages **libuv** for its single-threaded,
  **event-loop-based non‑blocking I/O**, enabling high concurrency without
  thread overhead.
- The relationship with V8 is close and symbiotic:

  - V8 executes JavaScript fast via JIT-compiled machine code.
  - Node.js embeds V8 and exposes additional APIs (e.g., `fs`, `http`) via C++
    bindings.
  - The event loop coordinates asynchronous operations executed in JavaScript by
    V8.

---

## JavaScript Execution Model

JavaScript is **single-threaded**, meaning it executes one instruction at a
time. But it appears to handle multiple tasks _in parallel_ due to:

- **Asynchronous APIs (provided by the browser or Node.js)**
- **The Event Loop**
- **Task queues** (microtasks & macrotasks)

| Concept    | Thread          | Priority  | Example                          |
| ---------- | --------------- | --------- | -------------------------------- |
| Call Stack | Main Thread     | Immediate | Regular JS code                  |
| Microtask  | Main Thread     | High      | `Promise.then`, `await`          |
| Macrotask  | Main Thread     | Lower     | `setTimeout`, I/O                |
| Web Worker | Separate Thread | Parallel  | Image processing, data crunching |

### Diagram: JavaScript Concurrency Flow

```txt
          [Call Stack] <-- main execution thread
               ↑
           [Event Loop]
            ↙       ↘
 [Microtask Queue]  [Macrotask Queue]
        ↑                   ↑
    Promise.then        setTimeout
    queueMicrotask      setInterval
```

### Call Stack

The **call stack** is the **execution context stack**. When a function is
invoked:

- It's pushed onto the stack.
- When it completes, it’s popped off.

```javascript
function greet() {
  console.log("Hello");
}
greet();
```

- `greet()` is pushed onto the stack.
- `console.log()` is called inside `greet()` and is also pushed.
- When `console.log()` finishes, it's popped.
- Then `greet()` is popped.

### Event Loop: The Coordinator

The **event loop** constantly checks:

1. Is the **call stack** empty?
2. Are there any **tasks in queues** (macrotasks, microtasks)?
3. If yes, it **dequeues the next task** and pushes it onto the call stack.

This is how asynchronous code is handled.

### Microtasks vs. Macrotasks

#### Microtasks

- High priority.
- Run **after the current task**, **before** the next macrotask.
- Include:

  - `Promise.then/catch/finally`
  - `queueMicrotask`
  - `MutationObserver`

#### Macrotasks

- Lower priority.
- Include:

  - `setTimeout`
  - `setInterval`
  - `setImmediate` (Node.js)
  - `MessageChannel`
  - DOM events
  - I/O (Node.js)

#### Execution Order Example

```javascript
console.log("Start");

setTimeout(() => console.log("Timeout"), 0);

Promise.resolve()
  .then(() => console.log("Microtask 1"))
  .then(() => console.log("Microtask 2"));

console.log("End");
```

**Output:**

```
Start
End
Microtask 1
Microtask 2
Timeout
```

1. `console.log("Start")` and `console.log("End")` are synchronous → stack
2. `Promise.then()` adds callbacks to **microtask queue**
3. `setTimeout()` adds callback to **macrotask queue**
4. Microtasks are drained **before** macrotasks.

### async/await

`async/await` is **syntactic sugar** over Promises that makes asynchronous code
look synchronous.

```javascript
async function fetchData() {
  console.log("1");
  await Promise.resolve();
  console.log("2");
}

fetchData();
console.log("3");
```

**Output:**

```
1
3
2
```

- `await` pauses function execution.
- The remainder is scheduled as a **microtask**.
- So `3` (outside async) runs before `2`.

### Web Workers: Multi-threading in JS

Web Workers allow JS to offload work to **background threads**.

- Communication is via `postMessage`.
- No access to DOM or `window`.
- Ideal for CPU-intensive tasks (image processing, parsing, etc.).

```javascript
// main.js
const worker = new Worker("worker.js");
worker.postMessage("start");

worker.onmessage = function (e) {
  console.log("From Worker:", e.data);
};
```

```javascript
// worker.js
self.onmessage = function (e) {
  if (e.data === "start") {
    let sum = 0;
    for (let i = 0; i < 1e8; i++) sum += i;
    self.postMessage(sum);
  }
};
```

---

## Web Workers

**Web Workers** allow you to run JavaScript code **in the background** on a
**separate thread**, so you can perform heavy computations **without blocking
the main UI thread**.

### Use case

- Data processing (e.g., image manipulation, crypto calculations)
- Long loops or math-heavy logic
- Avoiding UI "freezing" during intensive operations

### Limitations

- No DOM access inside the worker.
- No `alert`, `document`, `window`.
- Communication between the main thread and worker happens via **message
  passing** using `postMessage()` and `onmessage`.
- Must be served via **HTTP(S)** (not file://) due to security restrictions.

### Communication

- **Main thread → Worker**: `worker.postMessage(data)`
- **Worker → Main thread**: `self.postMessage(result)`
- **Worker listens**: `self.onmessage = function(...) { ... }`
- **Main thread listens**: `worker.onmessage = function(...) { ... }`

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Web Worker Demo</title>
  </head>
  <body>
    <h1>Web Worker Example</h1>
    <button onclick="startWorker()">Start Worker</button>
    <p id="result"></p>

    <script>
      let worker;

      function startWorker() {
        if (typeof Worker !== "undefined") {
          if (!worker) {
            worker = new Worker("worker.js");
          }

          worker.onmessage = function (event) {
            document.getElementById("result").textContent =
              "Worker says: " + event.data;
          };

          // Send message to worker
          worker.postMessage("start");
        } else {
          alert("Web Workers are not supported in your browser.");
        }
      }
    </script>
  </body>
</html>
```

```javascript
// worker.js

self.onmessage = function (event) {
  if (event.data === "start") {
    // Perform some heavy task (e.g., counting)
    let sum = 0;
    for (let i = 0; i < 1e8; i++) {
      sum += i;
    }
    // Send result back to main thread
    self.postMessage(sum);
  }
};
```

---

## JavaScript Garbage Collector (GC)

The GC automatically frees memory when objects are no longer reachable in the
program. JavaScript uses **mark-and-sweep**: it marks reachable objects starting
from roots (`window`, stack variables) and sweeps away the rest.

```javascript
let obj = { name: "David" };
obj = null; // No references, eligible for GC
```

### Interview Questions

#### How does JavaScript decide when to free memory?

When an object becomes unreachable (no references from root objects), GC removes
it in a later pass.

#### What’s a memory leak in JS?

Unused objects kept reachable (e.g., via closures, global vars, DOM references).

---

## Closures & Scope Chains

A closure is when a function "remembers" variables from its lexical scope even
after that scope has exited.

```javascript
function makeCounter() {
  let count = 0;
  return function () {
    return ++count;
  };
}
const counter = makeCounter();
console.log(counter()); // 1
console.log(counter()); // 2
```

### Interview Questions

#### How do closures cause memory leaks?

If a closure holds large data that’s no longer needed but still referenced.

#### How does the scope chain work?

Variables are looked up starting in the current scope, then outer scopes, up to
the global scope.

---

## Modules & Imports

ES modules use `import`/`export` for file-based modularity. They are static
(resolved at compile time), and run in strict mode.

```javascript
// math.js
export function add(a, b) {
  return a + b;
}

// main.js
import { add } from "./math.js";
console.log(add(2, 3));
```

### Interview Questions

#### Difference between CommonJS and ES Modules?

CommonJS is synchronous (`require`), ES modules are static and support
tree-shaking.

#### Can you `import` conditionally?

Yes, using `import()` (dynamic import) which returns a promise.

---

## Type Coercion & Equality

JavaScript converts values between types automatically (implicit coercion). `==`
uses coercion, `===` doesn’t.

```javascript
console.log(1 == "1"); // true
console.log(1 === "1"); // false
console.log([] == 0); // true ( [] → '' → 0 )
```

### Interview Questions

#### Why `[] == 0` is `true`?

`[]` → `''` → `0` when coerced to number.

#### When should you use `==`?

Generally avoid, unless you explicitly want coercion.

---

## WeakMap / WeakSet

They hold _weak_ references to keys, allowing GC to collect them if there are no
other references.

- `WeakMap`: keys must be objects. Values are arbitrary.
- `WeakSet`: holds objects without duplicates.

```javascript
let wm = new WeakMap();
let obj = {};
wm.set(obj, "secret");
obj = null; // GC can collect obj & its entry in wm
```

### Interview Questions

#### Why use WeakMap over Map?

For storing metadata tied to object lifecycle without preventing GC.

#### Can you iterate WeakMap?

No — keys are not enumerable.

---

## Symbol & Well-known Symbols

`Symbol()` creates unique identifiers. **Well-known symbols** are predefined
ones that let you customize object behavior (`Symbol.iterator`,
`Symbol.toStringTag`, etc.).

```javascript
const id = Symbol("id");
let user = { name: "Ali", [id]: 123 };

let arr = [1, 2, 3];
arr[Symbol.iterator] = function* () {
  yield 42;
};
console.log([...arr]); // [42]
```

### Interview Questions

#### Why use Symbols as object keys?

To avoid name collisions and keep keys hidden from normal iteration.

#### Give an example of a well-known symbol.

`Symbol.iterator` — defines default iteration behavior.

---

## Promises & Async/Await

### What is a Promise?

A **Promise** is an object representing the eventual completion (or failure) of
an asynchronous operation.

- **States**:

  1. `pending` → Initial state.
  2. `fulfilled` → Operation completed successfully.
  3. `rejected` → Operation failed.

```javascript
const promise = new Promise((resolve, reject) => {
  setTimeout(() => resolve("Done!"), 1000);
});

promise.then((result) => console.log(result)); // "Done!" after 1s
```

#### Execution Model (Promises)

1. **Creation**: Executor function runs immediately (synchronous start).
2. **Resolution/Rejection**: Happens later, asynchronously.
3. **Callbacks** in `.then()` and `.catch()` go into the **microtask queue** —
   meaning they run **after** the current synchronous code but **before**
   macrotasks like `setTimeout`.

#### Common Promise Methods

| Method                          | Description                               | Example                     |
| ------------------------------- | ----------------------------------------- | --------------------------- |
| `then(onFulfilled, onRejected)` | Runs when fulfilled or rejected           | `p.then(v => ...)`          |
| `catch(onRejected)`             | Error handling                            | `p.catch(e => ...)`         |
| `finally(onFinally)`            | Runs regardless of result                 | `p.finally(() => ...)`      |
| `Promise.all([...])`            | All must succeed or reject                | `Promise.all([p1, p2])`     |
| `Promise.allSettled([...])`     | Waits for all, regardless of success/fail | `Promise.allSettled([...])` |
| `Promise.race([...])`           | Resolves/rejects with first to finish     | `Promise.race([...])`       |
| `Promise.any([...])`            | Resolves with first success               | `Promise.any([...])`        |

### Async/Await

- **Syntactic sugar** for Promises.
- Makes asynchronous code look synchronous.
- An `async` function **always returns a Promise**.
- `await` pauses execution **inside the async function** until the Promise
  settles.

```javascript
async function getData() {
  try {
    const result = await fetch("https://api.example.com/data");
    return await result.json();
  } catch (err) {
    console.error(err);
  }
}

getData().then(console.log);
```

#### Execution Model (Async/Await)

- `await` doesn’t block the entire program — it suspends the **current async
  function** until the Promise resolves.
- Behind the scenes, `await` is just `.then()` with extra handling for
  `try/catch`.

### Scenarios & Best Practices

| Scenario                      | Promise         | Async/Await                |
| ----------------------------- | --------------- | -------------------------- |
| Multiple parallel async tasks | `Promise.all()` | `await Promise.all([...])` |
| Sequential async steps        | Chain `.then()` | Use multiple `await`       |
| Error handling                | `.catch()`      | `try/catch`                |
| Clean syntax for long chains  | ❌ Verbose      | ✅ Cleaner                 |

### Tricky Edge Cases

#### Promise executor runs immediately

```javascript
const p = new Promise((resolve) => {
  console.log("Executor runs immediately");
  resolve(42);
});
p.then(console.log);
```

#### Async function always returns a Promise

```javascript
async function f() {
  return 1;
}
f().then(console.log); // Logs 1, but as a Promise result
```

#### Microtasks vs Macrotasks

```javascript
setTimeout(() => console.log("Timeout"), 0);

Promise.resolve()
  .then(() => console.log("Promise 1"))
  .then(() => console.log("Promise 2"));

console.log("End");

// Output order:
// End
// Promise 1
// Promise 2
// Timeout
```

### Interview Questions

#### Why is `.then()` callback always async even if the Promise is already resolved?

Promise callbacks go into the **microtask queue** to ensure a consistent
execution order and avoid unexpected sync execution.

#### How does `await` differ from `yield` in generators?

`await` works only with Promises and automatically resumes execution when
resolved, while `yield` is manual — requires an iterator to resume.

#### What’s wrong with using `await` in a loop for multiple async operations?

```javascript
for (let url of urls) {
  await fetch(url); // Sequential, slow
}
```

This is sequential; better to use `Promise.all` for parallel execution.

#### What happens if you forget `await`?

You’ll get a Promise object instead of the resolved value — possibly leading to
logic errors.

---

## Data Fetching

### Fetch API (Modern)

```javascript
fetch("https://api.example.com/data")
  .then((response) => {
    if (!response.ok) {
      throw new Error("HTTP error " + response.status);
    }
    return response.json();
  })
  .then((data) => console.log(data))
  .catch((error) => console.error(error));
```

With `async/await`:

```javascript
async function fetchData() {
  try {
    const res = await fetch("https://api.example.com/data");
    if (!res.ok) throw new Error("HTTP error " + res.status);
    const data = await res.json();
    console.log(data);
  } catch (err) {
    console.error(err);
  }
}
```

### Request Cancellation (Fetch & Controller API)

The **Fetch API** doesn’t provide a built-in `.abort()` method, but cancellation
is made possible via the **AbortController API**, which is part of the DOM
standard and integrated with `fetch()`. `AbortController` is used to **signal
cancellation** to one or more operations (like `fetch()`) via an associated
`AbortSignal`.

- `controller.abort()` triggers cancellation.
- The `fetch` will throw an `AbortError`, which you should catch.
- The `signal` object is passed to the `fetch()` options.

```javascript
const controller = new AbortController();
const signal = controller.signal;

fetch("https://jsonplaceholder.typicode.com/posts", { signal })
  .then((response) => response.json())
  .then((data) => console.log(data))
  .catch((err) => {
    if (err.name === "AbortError") {
      console.log("Fetch aborted!");
    } else {
      console.error("Fetch error:", err);
    }
  });

// Cancel the fetch after 100ms
setTimeout(() => controller.abort(), 100);
```

#### Under the hood

- Fetch requests go through a **controller layer** managing lifecycle and
  resources.
- The request object has a **reference to the signal**.
- The network stack (e.g., via Blink or networking libraries like
  `net::URLLoader`) polls or subscribes to signal state.
- If `abort()` is triggered:

  - Ongoing TCP requests are forcibly closed.
  - Response stream is terminated.
  - Memory is cleaned up.
  - JS layer gets an `AbortError`.

### XHR (Legacy)

```javascript
const xhr = new XMLHttpRequest();
xhr.open("GET", "https://api.example.com/data", true);

xhr.onload = function () {
  if (xhr.status === 200) {
    const data = JSON.parse(xhr.responseText);
    console.log(data);
  } else {
    console.error("Error: " + xhr.status);
  }
};

xhr.onerror = function () {
  console.error("Request failed");
};

xhr.send();
```

### Interview Questions

#### When would you still use XHR?

- When you need **upload/download progress events**.
- When working on legacy projects with **IE 11 or older** support.

#### Why is Fetch preferred in modern apps?

- Cleaner syntax, better composability with Promises and `async/await`.
- Easier to work with in modern SPAs (e.g., React, Angular).
- Supports **streaming** and `AbortController`.

#### What are pitfalls of Fetch?

- Does **not reject** the promise on HTTP errors like 404/500 — must manually
  check `response.ok`.
- No built-in upload/download progress tracking (requires `ReadableStream` or
  third-party APIs).

---

## Object

A **JavaScript Object** is a **key-value data structure** used to store and
organize data. It’s the foundation of most things in JavaScript, including
functions, arrays, and even classes.

```javascript
const user = {
  name: "Alice",
  age: 30,
  greet: function () {
    console.log(`Hello, I'm ${this.name}`);
  },
};
```

### Object Properties

- **Key-value pairs** where keys are strings (or Symbols).
- Can be **data properties** or **accessor properties** (getters/setters).
- Stored in a **property descriptor** object with attributes like:

  - `value`
  - `writable`
  - `enumerable`
  - `configurable`

```javascript
Object.defineProperty(user, "role", {
  value: "admin",
  writable: false,
  enumerable: true,
  configurable: true,
});
```

### Object Methods

| Method                       | Purpose                        |
| ---------------------------- | ------------------------------ |
| `Object.keys(obj)`           | Returns array of keys          |
| `Object.values(obj)`         | Returns array of values        |
| `Object.entries(obj)`        | Returns key-value pairs        |
| `Object.assign(target, src)` | Copies properties              |
| `Object.freeze(obj)`         | Makes object immutable         |
| `Object.seal(obj)`           | Prevents adding/removing props |
| `obj.hasOwnProperty("key")`  | Checks `key` existence         |

### Prototype Chain & Inheritance

All objects inherit from `Object.prototype` by default unless created with
`Object.create(null)`.

```javascript
console.log({}.toString()); // [object Object]
```

Custom objects can inherit methods via prototypes:

```javascript
function Person(name) {
  this.name = name;
}
Person.prototype.sayHello = function () {
  return `Hi, I'm ${this.name}`;
};
```

### Interview Questions

#### What is the difference between `Object.create(null)` and `{}`?

- `{}` creates an object with a prototype chain (inherits from
  `Object.prototype`).
- `Object.create(null)` creates a truly **plain object** with no inherited
  properties.

#### How do you make an object immutable?

Use `Object.freeze(obj)`. It prevents:

- Adding/removing properties.
- Changing existing property values.
- Modifying descriptors.

```javascript
const obj = Object.freeze({ a: 1 });
obj.a = 2; // No effect
```

#### What's the difference between `Object.assign()` and spread `...`?

Both copy enumerable own properties:

- `Object.assign()` copies properties and returns the target object.
- Spread `...` syntax is cleaner but can't copy non-enumerables or preserve
  property descriptors.

```javascript
const obj1 = { a: 1 };
const obj2 = { b: 2 };

const merged = { ...obj1, ...obj2 }; // cleaner
```

#### What are getter and setter properties?

They allow **computed access** to object properties.

```javascript
const user = {
  firstName: "John",
  lastName: "Doe",
  get fullName() {
    return `${this.firstName} ${this.lastName}`;
  },
  set fullName(name) {
    [this.firstName, this.lastName] = name.split(" ");
  },
};
```

#### Explain object inheritance in JavaScript.

JavaScript uses **prototypal inheritance**. Objects inherit from other objects
via the `[[Prototype]]` internal link (set using `__proto__` or
`Object.setPrototypeOf()`).

#### How do you clone an object in JS?

> Better: use `structuredClone(obj)` (modern browsers).

- Shallow clone

  ```javascript
  const clone = { ...original };
  ```

- Deep clone (limited):

  ```javascript
  const deepClone = JSON.parse(JSON.stringify(original));
  ```

#### How to Check if an Object Has a Key

- Using `"key" in obj`

  Checks whether the key exists **anywhere in the object or its prototype
  chain**.

  ```javascript
  const obj = { name: "Alice" };
  console.log("name" in obj); // true
  ```

- Using `obj.hasOwnProperty("key")`

  Checks whether the key exists **only directly on the object** (not inherited).

  ```javascript
  console.log(obj.hasOwnProperty("name")); // true
  ```

- Using `Object.keys(obj).includes("key")`

  Less performant, returns true only if the key is **own and enumerable**.

---

## JavaScript Prototypes

Every JavaScript object has an internal link to another object called its
**prototype**. They define the **inheritance model** in JavaScript.

- When accessing a property or method:

  1. JavaScript looks at the object itself.
  2. If not found, it checks the object’s prototype (`__proto__` or
     `[[Prototype]]`).
  3. This continues recursively up the prototype chain.

### Setting and Getting Prototype

```javascript
const animal = {
  eats: true,
};

const rabbit = Object.create(animal);
rabbit.jumps = true;

console.log(rabbit.eats); // true (from prototype)
console.log(Object.getPrototypeOf(rabbit) === animal); // true
```

### Object Prototype Chain

```javascript
function Person(name) {
  this.name = name;
}
Person.prototype.greet = function () {
  return `Hi, I'm ${this.name}`;
};

const p = new Person("Bob");
console.log(p.greet()); // Hi, I'm Bob
```

Here:

- `p` has its own `name`
- `greet()` is accessed via `Person.prototype`

---

## What Does It Mean You Can `new` a Function in JavaScript?

In JS, **functions are special objects**. When a function is **called with
`new`**, JavaScript treats it as a **constructor call**:

- All functions in JS have an internal `[[Construct]]` method if they’re
  declared normally (not arrow functions).
- This makes them callable via `new`.
- Functions without `[[Construct]]` (like arrow functions or `class` static
  methods) **cannot** be called with `new`.

### Steps when you do `new Func()`

1. Create a new empty object: `{}`.
2. Link that object to `Func.prototype` via its internal `[[Prototype]]`.
3. Call `Func` with `this` bound to that new object.
4. If `Func` doesn’t return its own object, return the new object.

```javascript
function Car(brand) {
  this.brand = brand;
}

const myCar = new Car("Toyota");

console.log(myCar.brand); // Toyota
console.log(Object.getPrototypeOf(myCar) === Car.prototype); // true
```

---

## Classes

JavaScript introduced the `class` syntax in ES6 as **syntactic sugar** over its
existing prototype-based object model. Under the hood, nothing fundamentally
changes—it's still prototype inheritance at work.

```javascript
class Person {
  constructor(name) {
    this.name = name;
  }
  greet() {
    return `Hi, I'm ${this.name}`;
  }
}

class Student extends Person {
  constructor(name, id) {
    super(name);
    this.id = id;
  }
}
```

### Prototypes: The Real Backbone

- Each function has a `prototype` property used when creating instances.
- An object created via `new` links its internal `[[Prototype]]` (accessible via
  `__proto__` or `Object.getPrototypeOf`) to that prototype object.
- Method/property lookup walks up this prototype chain.

### Classes vs Prototype-Based Approach

Classes don't introduce new behavior—they merely wrap the classical prototype
mechanism into a cleaner syntax. You still get:

- **Static methods**: defined directly on the class/function
- **Instance methods**: defined on the prototype and inherited by instances

```javascript
class Foo {
  static staticMethod() {}
  instanceMethod() {}
}
```

Is equivalent to:

```javascript
function Foo() {}
Foo.staticMethod = function () {};
Foo.prototype.instanceMethod = function () {};
```

And instances use `instanceMethod` via the prototype chain, while `staticMethod`
is called on the class itself.

### Class methods vs Arrow functions vs Regular Functions

| Feature                      | `method() {}`                        | `method = () => {}`                                         | Regular Functions                         |
| ---------------------------- | ------------------------------------ | ----------------------------------------------------------- | ----------------------------------------- |
| Defined on Prototype         | Yes — shared across instances        | No — created individually per instance                      | No — new function per instance            |
| `this` Binding               | Dynamic—depends on how called        | Lexically fixed to instance (from outer scope)              | Dynamic—just like regular functions       |
| Supports `super`             | Yes — inherits from parent class     | No — can't use `super` properly                             | Yes, if method defined in prototype chain |
| Memory Efficiency            | High — one function shared per class | Low — each instance has its own copy                        | Low — same as arrow function fields       |
| Use as Constructor/Prototype | N/A — not functions themselves       | No — arrow functions have no `prototype` and can't be `new` | Same limitation for arrow-defined fields  |

### Interview Questions

#### What’s the difference between a class and an object in JavaScript?

- A **class** is syntactic sugar—a function under the hood.
- An **object** is an instance created with `new`, linked to the class's
  `prototype`.

#### How are class methods implemented?

- **Static methods** are assigned directly on the constructor function.
- **Instance methods** live on the `prototype`, shared across all instances.

#### Can you mix class syntax and manual prototype extension?

Yes. After defining:

```javascript
class Cat {}
Cat.prototype.meow = () => console.log("Meow");
```

This adds `meow` to all Cat instances—same as via class body but done manually.

#### Is class-based inheritance different from prototype inheritance?\*\*

No—class-based inheritance in JS is just a more readable wrapper around
prototype inheritance.

#### What are property descriptors in JS?

Property descriptors define how a property behaves. You can get them using
`Object.getOwnPropertyDescriptor(obj, key)`.

Attributes include:

- `value`
- `writable`
- `enumerable`
- `configurable`

#### What is the difference between `Object.create(null)` and `{}`?

- `{}` creates an object with a prototype chain (inherits from
  `Object.prototype`).
- `Object.create(null)` creates a truly **plain object** with no inherited
  properties.

Useful for dictionaries or maps where inherited properties could cause
conflicts.

#### How do you make an object immutable?

Use `Object.freeze(obj)`. It prevents:

- Adding/removing properties.
- Changing existing property values.
- Modifying descriptors.

```javascript
const obj = Object.freeze({ a: 1 });
obj.a = 2; // No effect
```

#### What's the difference between `Object.assign()` and spread `...`?

Both copy enumerable own properties:

- `Object.assign()` copies properties and returns the target object.
- Spread `...` syntax is cleaner but can't copy non-enumerables or preserve
  property descriptors.

```javascript
const obj1 = { a: 1 };
const obj2 = { b: 2 };

const merged = { ...obj1, ...obj2 }; // cleaner
```

#### What are getter and setter properties?

**Answer:**

```javascript
const user = {
  firstName: "John",
  lastName: "Doe",
  get fullName() {
    return `${this.firstName} ${this.lastName}`;
  },
  set fullName(name) {
    [this.firstName, this.lastName] = name.split(" ");
  },
};
```

They allow **computed access** to object properties.

#### Explain object inheritance in JavaScript

JavaScript uses **prototypal inheritance**. Objects inherit from other objects
via the `[[Prototype]]` internal link (set using `__proto__` or
`Object.setPrototypeOf()`).

#### How do you check if an object has a specific key

- Use `"key" in obj` for any key (own + inherited).
- Use `obj.hasOwnProperty("key")` for own keys only.

#### What is the prototype of an object?

It's another object from which the current object **inherits** methods and
properties.

#### What’s the difference between `__proto__`, `prototype`, and `[[Prototype]]`?

- `__proto__`: Legacy accessor for `[[Prototype]]`. Still widely supported.
- `prototype`: A property on **constructor functions**, used when creating new
  instances.
- `[[Prototype]]`: The internal slot representing the prototype chain. Not
  accessible directly, but can be read via `Object.getPrototypeOf(obj)`.

#### How does the prototype chain work, and why is it important?

When accessing a property on an object that doesn’t exist directly, JavaScript
traverses up `[[Prototype]]` chain until it either finds it or reaches `null`.
This chain enables **prototypal inheritance**, allowing objects to reuse methods
and properties defined higher in the chain. For example, an `Array` instance
accesses `filter()` via its prototype.

#### How would you implement inheritance using prototypes manually?

Using constructor functions and manually setting the prototype:

```javascript
function Animal(name) {
  this.name = name;
}
Animal.prototype.speak = () => console.log(this.name);

function Dog(name) {
  Animal.call(this, name);
}
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog;
```

This sets up prototypal inheritance such that `dog instanceof Animal` works
correctly.

#### Why is `hasOwnProperty` safer than `"key" in obj`

Because `"key" in obj` checks inherited properties too, which can cause bugs
when looping or validating object shape.

#### Can we override inherited methods?

Yes. You can override them directly on the object.

```javascript
const obj = Object.create({ greet: () => "Hi" });
obj.greet = () => "Hello";
console.log(obj.greet()); // "Hello"
```

---

## What is `this`?

`this` is a special keyword in JavaScript that **refers to the context in which
a function is executed** — not where it’s defined.

### General Rules for `this` binding

#### Default Binding (non–strict mode)

If a function is called without an explicit owner, `this` defaults to the
**global object** (`window` in browsers, `global` in Node.js).

```javascript
function foo() {
  console.log(this);
}
foo(); // window (in browsers, non-strict mode)
```

In **strict mode**:

```javascript
"use strict";
function foo() {
  console.log(this);
}
foo(); // undefined
```

#### Implicit Binding

When you call a function as a **property of an object**, `this` refers to that
object.

```javascript
const obj = {
  name: "David",
  greet: function () {
    console.log(`Hi, I am ${this.name}`);
  },
};
obj.greet(); // "Hi, I am David"
```

⚠ **Pitfall**: Losing implicit binding

```javascript
const greetFn = obj.greet;
greetFn(); // undefined (or "Hi, I am undefined") because `this` is lost
```

#### Explicit Binding (`call`, `apply`, `bind`)

You can explicitly set `this` using these methods.

```javascript
function greet(city) {
  console.log(`Hi, I am ${this.name} from ${city}`);
}
const person = { name: "David" };

greet.call(person, "Berlin"); // Hi, I am David from Berlin
greet.apply(person, ["Berlin"]); // same
const boundGreet = greet.bind(person, "Berlin");
boundGreet(); // Hi, I am David from Berlin
```

#### `new` Binding

When a function is called with `new`, a new object is created and set as `this`.

```javascript
function Person(name) {
  this.name = name;
}
const p = new Person("David");
console.log(p.name); // "David"
```

#### Arrow Functions

Arrow functions **do not have their own `this`** — they lexically inherit `this`
from their surrounding scope.

```javascript
const obj = {
  name: "David",
  greet: () => {
    console.log(this.name); // undefined, `this` is from global scope
  },
};
obj.greet();
```

Useful for preserving `this`:

```javascript
function Timer() {
  this.seconds = 0;
  setInterval(() => {
    this.seconds++;
    console.log(this.seconds);
  }, 1000);
}
new Timer();
```

### Special Cases / Edge Cases

#### Method extracted from object

```javascript
const obj = {
  name: "Test",
  sayName() {
    console.log(this.name);
  },
};
const fn = obj.sayName;
fn(); // undefined
```

#### Event Handlers

```javascript
document.querySelector("button").addEventListener("click", function () {
  console.log(this); // button element
});

document.querySelector("button").addEventListener("click", () => {
  console.log(this); // inherits from enclosing scope (likely window)
});
```

#### `this` in Class methods

```javascript
class Person {
  constructor(name) {
    this.name = name;
  }
  sayName() {
    console.log(this.name);
  }
}
const p = new Person("David");
p.sayName(); // "David"

const method = p.sayName;
method(); // undefined (method lost its binding)
```

### Interview Questions

#### Quick Interview Traps

```javascript
var name = "Global";
const obj = {
  name: "Obj",
  arrow: () => console.log(this.name),
  method() {
    console.log(this.name);
  },
};
obj.arrow(); // undefined (arrow has no own `this`)
obj.method(); // "Obj"
```

#### Explain the different ways `this` gets bound in JavaScript.

- **Global/Function Call:** In non-strict mode, `this` is the global object; in
  strict mode, it's `undefined`.
- **Method Call:** `obj.method()` → `this` is `obj`.
- **Constructor (`new`):** `this` points to the newly created object.
- **Explicit Binding:** `.call()`, `.apply()`, `.bind()` explicitly set `this`.
- **Arrow Functions:** `this` is lexically inherited from the enclosing scope.

#### What are common pitfalls with `this` in callbacks or event handlers?

Losing `this` context happens when passing methods as callbacks:

```javascript
const obj = {
  name: "Alice",
  greeting() {
    console.log(this.name);
  },
};
setTimeout(obj.greeting, 0); // `this` lost → undefined or global
```

Fixes include `.bind(obj)`, arrow function wrappers, or caching the context
(`const self = this`).

---

## IIFE

An IIFE is a fundamental idiom for avoiding polluting the global namespace,
enabling local variable scoping and immediate execution of logic, useful in
module pattern implementation and self-contained initializations. It is widely
used in pre-ES6 code and remains important for encapsulating logic, particularly
in frameworks and build scripts.

---

## Generator Functions

A **generator function** is a special type of function in JavaScript that can be
paused and resumed. They are defined with `function*` syntax and use `yield` to
_return a value temporarily_ while keeping the function’s internal state.

```javascript
function* exampleGenerator() {
  yield 1;
  yield 2;
  yield 3;
}

const gen = exampleGenerator();
console.log(gen.next()); // { value: 1, done: false }
console.log(gen.next()); // { value: 2, done: false }
console.log(gen.next()); // { value: 3, done: false }
console.log(gen.next()); // { value: undefined, done: true }
```

### How `yield` works

- `yield <expression>`: pauses execution, returns `<expression>`.
- `generator.next(<value>)`: resumes execution, and `<value>` becomes the result
  of the last `yield` inside the function.

```javascript
function* greet() {
  const name = yield "What's your name?";
  yield `Hello, ${name}!`;
}

const g = greet();
console.log(g.next()); // Ask: "What's your name?"
console.log(g.next("Alice")); // Respond: "Hello, Alice!"
```

### Execution Model vs Normal Functions

| Normal Functions               | Generator Functions                            |
| ------------------------------ | ---------------------------------------------- |
| Run from start to finish once. | Can pause (`yield`) and resume multiple times. |
| Return a single value.         | Return multiple values over time.              |
| Call returns the result.       | Call returns an **iterator** object.           |

### Usage Scenarios

#### Lazy Evaluation (on-demand data)

```javascript
function* infiniteCounter() {
  let i = 0;
  while (true) yield i++;
}

const counter = infiniteCounter();
console.log(counter.next().value); // 0
console.log(counter.next().value); // 1
console.log(counter.next().value); // 2
```

Useful for **infinite streams** or **large data sets** where you don’t want to
load everything at once.

#### Custom Iterators

You can make any object iterable:

```javascript
const myCollection = {
  items: ["apple", "banana", "cherry"],
  *[Symbol.iterator]() {
    for (const item of this.items) yield item;
  },
};

for (const fruit of myCollection) {
  console.log(fruit); // apple, banana, cherry
}
```

#### Bidirectional Communication

You can send data _into_ a generator at each `next()` call.

```javascript
function* calculator() {
  let result = yield;
  while (true) {
    const nextValue = yield result;
    result += nextValue;
  }
}

const calc = calculator();
calc.next(); // start
console.log(calc.next(5).value); // 5
console.log(calc.next(3).value); // 8
```

### Interview Questions

#### What happens if you `return` inside a generator?

It ends the generator, setting `done: true`.

```javascript
function* g() {
  yield 1;
  return 2;
  yield 3; // never reached
}
console.log([...g()]); // [1]
```

#### Can you `throw` into a generator?

Yes — `iterator.throw(error)` resumes execution by throwing inside the
generator.

```javascript
function* g() {
  try {
    yield 1;
  } catch (e) {
    yield `Caught: ${e}`;
  }
}
const it = g();
console.log(it.next()); // {value: 1, done: false}
console.log(it.throw("Oops")); // {value: 'Caught: Oops', done: false}
```

#### How are generators related to async functions?

`async function` is essentially **a generator + a promise scheduler** that
automatically handles `.next()` calls.

---

## Arrays

### push

```javascript
arr.push("foo");
// [      | + ]
```

### pop

```javascript
arr.pop();
// [      | - ]
```

### shift

```javascript
arr.shift();
// [ - |      ]
```

### unshift

```javascript
arr.unshift("foo");
// [ + |      ]
```

### concat

```javascript
["foo", "bar"].concat(["qux"]);
// ["foo", "bar", "qux"]
```

### includes

```javascript
["foo", "bar"].includes("bar");
// true
```

### indexOf

```javascript
["foo", "bar"].indexOf("bar");
// 1

["foo", "bar"].indexOf("qux");
// -1
```

### reverse

> The `reverse()` method reverses an array **in place**.

```javascript
["foo", "bar"].reverse();
// ["bar", "foo"]
```

### join

```javascript
["Foo", "Bar", "Qux"].join();
// "Foo,Bar,Qux"

["Foo", "Bar", "Qux"].join("");
// "FooBarQux"

["Foo", "Bar", "Qux"].join("-");
// "Foo-Bar-Qux"
```

### slice

```javascript
// make a copy of an array
let foo = [1, 2, 3];
let bar = foo;
let qux = foo.slice();

foo === bar;
// true <- referencing the same object in memory

foo === qux;
// false <- copy
```

```javascript
["foo", "bar", "qux"].slice(1);
// ["bar", "qux"]

["foo", "bar", "qux"].slice(1, 2);
// ["bar"]

["foo", "bar", "qux"].slice(-1);
// ["qux"]
```

### splice

> The `splice()` method changes the contents of an array by removing or
> replacing existing elements and/or adding new elements in **place**.

```javascript
// arr.splice(start, ?deleteCount, ...items);
// returns the deleted items, if any

let foo = ["foo", "bar", "qux"];
foo.splice(1, 0, "baz");
// return: []
// foo: ["foo", "baz", "bar", "qux"]

foo.splice(2, 1);
// return: ["bar"]
// foo: ["foo", "baz", "qux"]

foo.splice(0, 2, "FOO", "BAZ");
// return: ["foo", "baz"]
// foo: ["FOO", "BAZ", "qux"]
```

### fill

```javascript
[1, 2, 3, 4, 5].fill(6);
// [6, 6, 6, 6, 6]

[1, 2, 3, 4, 5].fill(6, 2);
// [1, 2, 6, 6, 6]

[1, 2, 3, 4, 5].fill(6, 2, 4);
// [1, 2, 6, 6, 5]
```

### sort

> The `sort()` method sorts the elements of an array **in place** and returns
> the sorted array.

```javascript
["foo", "bar", "qux"].sort();
// ["bar", "foo", "qux"]

[1, 33, 15, 100].sort();
// [1, 100, 15, 33]

[1, 33, 15, 100].sort((a, b) => a - b);
// [1, 15, 33, 100]
```

### forEach

```javascript
["foo", "bar", "qux"].forEach((currentValue) => {});

["foo", "bar", "qux"].forEach((currentValue, index, array) => {});
```

### map

```javascript
let foo = ["foo", "bar", "qux"].map((currentValue) => {
  return currentValue.toUpperCase();
});
// foo: ["FOO", "BAR", "QUX"]

let foo = ["foo", "bar", "qux"].map((currentValue, index, array) => {});
```

### find

> Returns the value of the **first element** in the array that satisfies the
> provided testing function. Otherwise, `undefined` is returned.

```javascript
["foo", "bar"].find((element) => {
  return element === "bar";
});
// "bar"

["foo", "bar"].find((element) => {
  return element === "qux";
});
// undefined

["foo", "bar"].find((element, index, array) => {});
```

### filter

> Returns a new array with the elements that pass the test. If no elements pass
> the test, an empty array will be returned.

```javascript
["foo", "bar", "bar baz"].filter((currentValue) => {
  return currentValue.includes("bar");
});
// ["bar", "bar baz"]

["foo", "bar"].filter((currentValue) => {
  return currentValue.includes("qux");
});
// []

["foo", "bar"].filter((currentValue, index, array) => {});
```

### every

> Returns `true` if the callback function returns a truthy value for **every**
> array element. Otherwise, `false`.

```javascript
[10, 20, 30].every((element) => {
  return element > 0;
});
// true

["foo", "bar", "baz"].every((element) => {
  return element.includes("a");
});
// false

["foo", "bar"].every((element, index, array) => {});
```

### some

> Returns `true` if the callback function returns a truthy value for **at least
> one** element in the array. Otherwise, `false`.

```javascript
[1, 3, 5, 7].some((element) => {
  return element > 4;
});
// true

[1, 3, 5, 7].some((element) => {
  return element > 11;
});
// false

["foo", "bar"].some((element, index, array) => {});
```

### reduce

> If no `initialValue` given, `accumulator` will be set to the first element.

```javascript
[1, 2, 3, 4].reduce((accumulator, currentValue) => {
  return accumulator + currentValue;
}, 0);
// 10

[1, 2, 3, 4].reduce((accumulator, currentValue) => {
  return currentValue % 2 === 0 ? accumulator + currentValue : accumulator;
}, 0);
// 6

[1, 2, 3].reduce((accumulator, currentValue, index, array) => {}, initialValue);
```

---

## String search

### includes

```javascript
"foo bar qux".includes("bar");
/* true */

"foo bar qux".includes("bar", 5);
/* false */
```

### endsWith

```javascript
"foo bar".endsWith("bar");
/* true */
```

### startsWith

```javascript
"foo bar".startsWith("foo");
/* true */
```

### indexOf

```javascript
"foo bar".indexOf("bar");
/* 4 */

"foo bar".indexOf("qux");
/* -1 */
```

### lastIndexOf

```javascript
"foo bar bar".lastIndexOf("bar");
/* 8 */

"foo bar bar".lastIndexOf("qux");
/* -1 */
```

### test

> TODO

[RegExp.test](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp/test)

```javascript

```

### search

> TODO

[String.search](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/search)

```javascript

```

### match

> TODO

[String.match](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/match)

```javascript

```

### matchAll

> TODO

[String.matchAll](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/matchAll)

```javascript

```

## String manipulation

### concat

```javascript
"foo".concat(" ", "bar", " ", "qux");
/* "foo bar qux" */
```

### padEnd

```javascript
"foo".padEnd(7, ".");
/* "foo...." */
```

### padStart

```javascript
"foo".padStart(7, ".");
/* "....foo" */
```

### repeat

```javascript
"z".repeat(3);
/* "zzz" */
```

### replaceAll

[replaceAll](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/replaceAll)

```javascript
"The quick brown fox jumps over the lazy dog".replaceAll("dog", "cat");
/* "The quick brown fox jumps over the lazy cat" */

"The quick brown fox jumps over the lazy dog".replaceAll(/dog/gi, "cat");
/* "The quick brown fox jumps over the lazy cat" */
```

### slice

```javascript
"foo bar qux".slice(4);
/* "bar qux" */

"foo bar qux".slice(4, 7);
/* "bar" */

"foo bar qux".slice(-3);
/* "qux" */
```

### split

```javascript
"foo bar qux".split();
/* ["foo bar qux"] */

"foo bar qux".split("");
/* ["f", "o", "o", " ", "b", "a", "r", " ", "q", "u", "x"] */

"foo bar qux".split(" ");
/* ["foo", "bar", "qux"] */

"foo bar qux".split(" ", 2);
/* ["foo", "bar"] */
```

### toLowerCase

```javascript
"FOO".toLowerCase();
/* "foo" */
```

### toUpperCase

```javascript
"foo".toUpperCase();
/* "FOO" */
```

### trim

```javascript
"  foo  ".trim();
/* "foo" */

"  foo  ".trimStart();
/* "foo  " */

"  foo  ".trimEnd();
/* "  foo" */
```

---

## Managing events

```javascript
function foo(event) {}

target.addEventListener("click", foo);

target.removeEventListener("click", foo);
```

## Event types

[Event reference](https://developer.mozilla.org/en-US/docs/Web/Events)

### Window events

#### load

> The `load` event is fired when the whole page has loaded, including all
> dependent resources such as stylesheets and images.

```javascript
window.addEventListener("load", foo);
```

#### unload

> The `unload` event is fired when the document or a child resource is being
> unloaded.

```javascript
window.addEventListener("unload", foo);
```

### Network events

#### online

> The `online` event of the `Window` interface is fired when the browser has
> **gained** access to the network.

```javascript
window.addEventListener("online", foo);
```

#### offline

> The `offline` event of the `Window` interface is fired when the browser has
> **lost** access to the network.

```javascript
window.addEventListener("offline", foo);
```

### Focus events

#### focus

> The `focus` event fires when an element has received focus.

```javascript
target.addEventListener("focus", foo, true);
```

#### blur

> The `blur` event fires when an element has lost focus.

```javascript
target.addEventListener("blur", foo, true);
```

### Viewport events

#### requestFullscreen

```javascript
element.requestFullscreen().catch((error) => {});

const fullscreen_element = document.fullscreenElement;
// fullscreen_element <- if already in fullscreen mode
// null <- if not

// exit fullscreen mode
document
  .exitFullscreen()
  .then(foo)
  .catch((error) => {});
```

#### fullscreenchange

```javascript
target.addEventListener("fullscreenchange", foo);
```

#### fullscreenerror

```javascript
target.addEventListener("fullscreenerror", foo);
```

#### resize

```javascript
window.addEventListener("resize", foo);
```

#### scroll

```javascript
document.addEventListener("scroll", foo);
```

### Keyboard events

#### keydown

> The `keydown` event is fired when a key is pressed.

```javascript
target.addEventListener("keydown", foo);
```

#### keyup

> The `keyup` event is fired when a key is released.

```javascript
target.addEventListener("keyup", foo);
```

#### KeyboardEvent

[KeyboardEvent](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent)

##### Modifier keys

```javascript
// true <- if held down
// false <- else
const altKeyState = KeyboardEvent.altKey;
const ctrlKeyState = KeyboardEvent.ctrlKey;
const shiftKeyState = KeyboardEvent.shiftKey;
const metaKeyState = KeyboardEvent.metaKey;

// true <- if held down such that it is automatically repeating
// false <- else
const repeatState = KeyboardEvent.repeat;
```

##### Key codes

> The `KeyboardEvent.code` property represents a physical key on the keyboard.
> This property is useful when you want to handle keys based on their physical
> positions on the input device rather than the characters associated with those
> keys.

> To determine what character corresponds with the key event, use the
> `KeyboardEvent.key` property instead.

[KeyboardEvent: code
values](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/code/code_values)

```javascript
KeyboardEvent.code;
// "KeyA"
```

### Mouse events

[MouseEvent](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent)

| Property  | Description                                        |
| --------- | -------------------------------------------------- |
| `pageX`   | The **X** coordinate relative to the **page**.     |
| `pageY`   | The **Y** coordinate relative to the **page**.     |
| `clientX` | The **X** coordinate relative to the **viewport**. |
| `clientY` | The **Y** coordinate relative to the **viewport**. |

| Property   | Description                                   |
| ---------- | --------------------------------------------- |
| `altKey`   | Returns `true` if the **alt** key was down.   |
| `ctrlKey`  | Returns `true` if the **ctrl** key was down.  |
| `shiftKey` | Returns `true` if the **shift** key was down. |
| `metaKey`  | Returns `true` if the **meta** key was down.  |

#### click

> A pointing device button has been pressed and released.

```javascript
target.addEventListener("click", foo);
```

#### contextmenu

> The `contextmenu` event fires when the user attempts to open a context menu.

```javascript
target.addEventListener("contextmenu", (event) => {
  event.preventDefault(); // disable context menu
  // event logic
});
```

#### dblclick

> The `dblclick` event fires when a pointing device button (such as a mouse's
> primary button) is double-clicked

```javascript
target.addEventListener("dblclick", foo);
```

#### mousedown

> This differs from the `click` event in that `click` is fired after a full
> click action occurs; that is, the mouse button is pressed and released while
> the pointer remains inside the same element. `mousedown` is fired the moment
> the button is initially pressed.

```javascript
target.addEventListener("mousedown", foo);
```

#### mouseup

> The `mouseup` event is fired at an `Element` when a button on a pointing
> device (such as a mouse or trackpad) is released while the pointer is located
> inside it.

```javascript
target.addEventListener("mouseup", foo);
```

#### wheel

> The `wheel` event fires when the user rotates a wheel button on a pointing
> device (typically a mouse).

```javascript
target.addEventListener("wheel", (event) => {
  event.preventDefault(); // disable page scroll
  const delta = event.deltaY; // vertical scroll amount
  // event logic
});
```

#### mousemove

> The `mousemove` event is fired at an `Element` when a pointing device (usually
> a mouse) is moved while the cursor's hotspot is inside it.

```javascript
target.addEventListener("mousemove", foo);
```

#### mouseover

> The `mouseover` event is fired at an `Element` when a pointing device (such as
> a mouse or trackpad) is used to move the cursor onto the element or one of its
> child elements.

```javascript
target.addEventListener("mouseover", foo);
```

#### mouseleave

> `mouseleave` is fired when the pointer has exited the element and all of its
> descendants, whereas `mouseout` is fired when the pointer leaves the element
> or leaves one of the element's descendants (even if the pointer is still
> within the element).

```javascript
target.addEventListener("mouseleave", foo);
```

### Drag & Drop events

| Property    | Description                                                                                              |
| ----------- | -------------------------------------------------------------------------------------------------------- |
| `dragstart` | The user starts dragging an element or text selection.                                                   |
| `dragend`   | A drag operation is being ended.                                                                         |
| `dragover`  | An element or text selection is being dragged over a valid drop target (fired continuously every 350ms). |
| `dragenter` | A dragged element or text selection enters a valid drop target.                                          |
| `dragleave` | A dragged element or text selection leaves a valid drop target.                                          |
| `drop`      | An element is dropped on a valid drop target.                                                            |

#### Element drag and drop

[Drag Event
Example](https://developer.mozilla.org/en-US/docs/Web/API/Document/drag_event)

```html
<div class="dropzone">
  <div class="draggable" draggable="true">This div is draggable</div>
</div>
<div class="dropzone"></div>
<div class="dropzone"></div>
<div class="dropzone"></div>
```

```css
.draggable {
  width: 200px;
  height: 20px;
  text-align: center;
  background: white;
}

.dropzone {
  width: 200px;
  height: 20px;
  background: blueviolet;
  margin-bottom: 10px;
  padding: 10px;
}
```

```javascript
let dragged;

/* draggable target events */
document.addEventListener("dragstart", (event) => {
  dragged = event.target;
  event.target.style.opacity = 0.3;
});

document.addEventListener("dragend", (event) => {
  event.target.style.opacity = "";
});

/* drop target events */
document.addEventListener("dragover", (event) => {
  // prevent default to allow drop
  event.preventDefault();
});

document.addEventListener("dragenter", (event) => {
  if (event.target.className == "dropzone") {
    event.target.style.background = "purple";
  }
});

document.addEventListener("dragleave", (event) => {
  if (event.target.className == "dropzone") {
    event.target.style.background = "";
  }
});

document.addEventListener("drop", (event) => {
  // prevent default action (open as link for some elements)
  event.preventDefault();

  if (event.target.className == "dropzone") {
    event.target.style.background = "";
    dragged.remove();
    event.target.append(dragged);
  }
});
```

#### File drag and drop

```javascript
fileDropTarget.addEventListener("dragover", (event) => {
  // prevent default to allow drop
  event.preventDefault();
});

fileDropTarget.addEventListener("drop", (event) => {
  // prevent default action (open as link for some elements)
  event.preventDefault();

  console.log(event.dataTransfer.files);
});
```

#### Dragging elements

[What is `this` in event
listeners?](https://metafizzy.co/blog/this-in-event-listeners/)

```html
<div class="draggable"></div>
```

```css
.draggable {
  display: inline-block;
  position: relative;
  width: 150px;
  height: 150px;
  background: #19f;
  border-radius: 8px;
  cursor: move;
}
```

```javascript
var dragElem = document.querySelector(".draggable");

var x = 0;
var y = 0;
var dragStartX, dragStartY, pointerDownX, pointerDownY;

dragElem.addEventListener("mousedown", function (event) {
  // keep track of start positions
  dragStartX = x;
  dragStartY = y;
  pointerDownX = event.pageX;
  pointerDownY = event.pageY;
  // add move & up events
  window.addEventListener("mousemove", onmousemove);
  window.addEventListener("mouseup", onmouseup);
});

function onmousemove(event) {
  // how much has moved
  var moveX = event.pageX - pointerDownX;
  var moveY = event.pageY - pointerDownY;
  // add movement to position
  x = dragStartX + moveX;
  y = dragStartY + moveY;
  // position element
  dragElem.style.left = x + "px";
  dragElem.style.top = y + "px";
}

function onmouseup() {
  // remove move & up events
  window.removeEventListener("mousemove", onmousemove);
  window.removeEventListener("mouseup", onmouseup);
}
```

---

## DOM Element Selection

### getElementById

> Returns an `Element` object describing the DOM element object matching the
> specified ID, or `null` if no matching element was found in the document.

> Unlike some other element-lookup methods such as `Document.querySelector()`
> and `Document.querySelectorAll()`, `getElementById()` is only available as a
> method of the global `document` object, and **not** available as a method on
> all element objects in the DOM. Because ID values must be unique throughout
> the entire document, there is no need for "local" versions of the function.

```javascript
const element = document.getElementById("foo");
```

```html
<div id="foo">SELECTED</div>
```

### getElementsByClassName

```javascript
const elements = document.getElementsByClassName("foo");
```

```html
<div class="foo">SELECTED</div>
<div class="foo bar">SELECTED</div>
```

### querySelectorAll

```javascript
const elementById = document.querySelector("#foo");
const elementsByClass = document.querySelectorAll("p, .bar");
```

```html
<div id="foo">SELECTED</div>
<p>SELECTED</p>
<div class="bar">SELECTED</div>
```

### Node tree traversal

```javascript
const parentElement = element.parentElement;
const children = element.children;

const firstElementChild = element.firstElementChild;
const lastElementChild = element.lastElementChild;

const nextElementSibling = element.nextElementSibling;
const previousElementSibling = element.previousElementSibling;
```

## Manipulation

### createElement

```javascript
const element = document.createElement("div");
```

### remove

```javascript
element.remove();
```

### append

```javascript
const parent = document.getElementById("foo");
const child = document.createElement("div");

parent.append(child);
```

### prepend

> Inserts a set of `Node` objects or `DOMString` objects **before the first
> child** of the `ParentNode`.

```javascript
parent.prepend(child);
```

### innerText

```javascript
div.innerText;
// "bold"
```

```html
<div><b>bold</b></div>
```

### innerHTML

```javascript
div.innerHTML;
// "<b>bold</b>"
```

```html
<div><b>bold</b></div>
```

### insertAdjacentHTML

```javascript
div.insertAdjacentHTML("beforeend", "<p>foo</p>");
```

```html
<!-- beforebegin -->
<div>
  <!-- afterbegin -->
  <!-- beforeend -->
</div>
<!-- afterend -->
```

### getAttribute

```javascript
input.getAttribute("type");
// "email"

input.getAttribute("foo");
// null <- doesn't exists

input.getAttribute("required");
// "" <- is set

input.toggleAttribute("required");

input.setAttribute("type", "radio");
```

```html
<input type="email" required />
```

### classList

```javascript
div.classList;
// ["foo", "bar"]

div.classList.add("qux");
// ["foo", "bar", "qux"]

div.classList.remove("qux");
// ["foo", "bar"]

div.classList.toggle("foo");
// ["bar"]
```

```html
<div class="foo bar"></div>
```

### style

```javascript
div.setAttribute("style", "color:red; padding:0.3rem;");

div.style.color = "blue";
```

---

## Storage

### LocalStorage

**Persists after closing the browser**

```javascript
// Persists till clearing the browser cache
localStorage.setItem("foo", "bar");
const result = localStorage.getItem("foo"); // bar

localStorage.removeItem("foo"); // deletes foo
localStorage.clear(); // clears everything
```

### Session Storage

**Get cleared after closing the tab**

```javascript
// Gets cleared when closing the tab
sessionStorage.setItem("foo", "bar");
const result = sessionStorage.getItem("foo"); // bar
```

### Cookie

```javascript
document.cookie = "foo=bar";
document.cookie = "qux=baz";

console.log(document.cookie);
// "foo=bar; qux=baz"

// expires in 10s
document.cookie = "foo=bar; max-age=10";
```

### IndexedDB

[localForage](https://localforage.github.io/localForage/)
