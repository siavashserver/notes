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

