---
title: Threading
---

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
