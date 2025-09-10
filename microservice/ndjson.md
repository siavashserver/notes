---
title: NDJSON
---

## Introduction

Newline-Delimited JSON (NDJSON) is a format where each line is a standalone JSON
object.  
It shines in streaming scenarios because you can parse and act on each record as
soon as it arrives, instead of waiting for a full JSON array.

- Incremental parsing: process items one by one.
- Lower memory overhead: no need to buffer full payloads.
- Simple protocol: split on `\n`, then `JSON.parse`.

## ASP.NET Core: NDJSON Endpoint

```csharp
using System.Text.Json;
using Microsoft.AspNetCore.Mvc;

[ApiController]
[Route("api/[controller]")]
public class StreamController : ControllerBase
{
    [HttpGet("ndjson")]
    public async Task StreamNdJson(CancellationToken ct)
    {
        Response.ContentType = "application/x-ndjson";

        for (int i = 1; i <= 5; i++)
        {
            var record = new { Id = i, Time = DateTime.UtcNow };
            var json   = JsonSerializer.Serialize(record);

            // Write a line and flush immediately
            await Response.WriteAsync(json + "\n", ct);
            await Response.Body.FlushAsync(ct);

            await Task.Delay(1000, ct);
        }
    }
}
```

## Frontend: Pure JavaScript with Fetch

```javascript
// Set up an AbortController for timeout/cancellation
const controller = new AbortController();
const timeoutId = setTimeout(() => controller.abort(), 30000);

fetch("/api/stream/ndjson", { signal: controller.signal })
  .then((response) => {
    const reader = response.body.getReader();
    const decoder = new TextDecoder();
    let buffer = "";

    // Recursive chunk reader
    function readLoop() {
      return reader.read().then(({ done, value }) => {
        if (done) {
          console.log("⏹️ Stream ended");
          return;
        }

        buffer += decoder.decode(value, { stream: true });
        const lines = buffer.split("\n");
        buffer = lines.pop(); // keep incomplete line

        for (const line of lines) {
          if (!line) continue;
          try {
            const obj = JSON.parse(line);
            console.log("📥 Received:", obj);
          } catch (err) {
            console.error("❌ Parse error:", err);
          }
        }
        return readLoop();
      });
    }

    return readLoop();
  })
  .catch((err) => {
    if (err.name === "AbortError") {
      console.error("⏲️ Request timed out");
    } else {
      console.error("🚨 Connection error:", err);
    }
  })
  .finally(() => clearTimeout(timeoutId));
```
