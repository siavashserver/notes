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
