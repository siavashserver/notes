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

```js
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

```js
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

```js
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

```js
// main.js
const worker = new Worker("worker.js");
worker.postMessage("start");

worker.onmessage = function (e) {
  console.log("From Worker:", e.data);
};
```

```js
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

```js
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
