---
title: gRPC
---

## Introduction

gRPC is a high-performance, open-source RPC framework developed by Google. It
uses HTTP/2 for transport, Protocol Buffers (protobuf) for efficient binary
serialization, and auto-generates strongly-typed client/server code from
`.proto` files.

- High throughput, low latency (binary vs JSON)
- Built-in support for unary & streaming (client, server, bi-directional)
- Strong API contracts, polyglot support, ideal for microservices

## gRPC RPC types

- Unary RPC: single request, single response
- Server streaming: one request, multiple responses
- Client streaming: multiple requests, one response
- Bidirectional streaming: full duplex

## HTTP/2 vs HTTP/1.1

- Multiplexing: multiple streams per TCP connection vs one per connection in
  HTTP/1.1
- Prioritization & server push: enhances performance
- Header compression: HPACK in HTTP/2 significantly reduces overhead
- Binary framing: efficient, structured message framing

## gRPC vs REST

| Feature   | REST (HTTP/1.1/2 + JSON)                | gRPC (HTTP/2 + Protobuf)                            |
| --------- | --------------------------------------- | --------------------------------------------------- |
| Protocol  | HTTP/1.1 (mostly), HTTP/2 (optional)    | HTTP/2 only                                         |
| Format    | JSON (text, verbose, human-readable)    | Protobuf (binary, compact, fast)                    |
| API style | Resource-focused with URLs & verbs      | Service-oriented RPC                                |
| Streaming | Not native (requires WebSocket/polling) | Native support for all streaming modes              |
| Tooling   | Lightweight, language agnostic          | Strong typing, code-gen, tight schema coupling      |
| Use cases | Public APIs, browser compatibility      | Internal services, high performance, real-time data |

---

## Protobuf

### File Declaration

Every `.proto` file begins by specifying syntax version:

```proto
syntax = "proto3";
package my_package;   // optional, avoids naming conflicts
```

### Messages & Fields

Define structured data with `message` blocks:

```proto
message Person {
  string name = 1;
  int32 id = 2;
  string email = 3;
}
```

Field types include basic scalar types (`int32`, `int64`, `string`, `bool`,
`bytes`) and composite types (nested messages, enums, maps). Each field has a
unique _tag number_ (1–2³²‑1), reserving 19000–19999.

By default in proto3:

- Fields are optional and have default **zero** values (e.g. `int=0`,
  `string=""`, `bool=false`).
- `optional` keyword is implicit; `required` is not supported.

### Repeated Fields

Use repeated to declare lists:

```proto
message Book {
  repeated string authors = 1;
  repeated Person contributors = 2;
}
```

### Enums

```proto
enum PhoneType {
  PHONE_TYPE_UNSPECIFIED = 0;  // must start at zero
  PHONE_TYPE_MOBILE = 1;
  PHONE_TYPE_HOME = 2;
}
```

### Nested Messages & Enums

```proto
message Person {
  message PhoneNumber {
    string number = 1;
    PhoneType type = 2;
  }
  repeated PhoneNumber phones = 4;
  enum PhoneType { ... }
}
```

### Maps

```proto
map<string, Project> projects = 3;
```

### Oneof Fields

Use `oneof` when only one of multiple possible fields should be set:

```proto
message Student {
  oneof id_card {
    string student_card_number = 5;
    string national_id = 6;
  }
}
```

### Services

```proto
service MyService {
  rpc GetPerson(GetPersonRequest) returns (GetPersonResponse);
}
```

---

## Sample Code

```proto
syntax = "proto3";
service MyService {
  rpc Unary (Req) returns (Res);
  rpc ServerStream (Req) returns (stream Res);
  rpc ClientStream (stream Req) returns (Res);
  rpc BidiStream (stream Req) returns (stream Res);
}
message Req { string msg = 1; }
message Res { string msg = 1; }
```

### Unary

#### Client

```csharp
var client = new MyService.MyServiceClient(channel);
var resp = await client.UnaryAsync(new Req { Msg = "Hello" });
Console.WriteLine(resp.Msg);
```

#### Server

```csharp
var client = new MyService.MyServiceClient(channel);
var resp = await client.UnaryAsync(new Req { Msg = "Hello" });
Console.WriteLine(resp.Msg);
```

### Server Streaming

#### Client

```csharp
var call = client.ServerStream(new Req { Msg = "Hi" });
await foreach (var r in call.ResponseStream.ReadAllAsync())
    Console.WriteLine(r.Msg);
```

#### Server

```csharp
public override async Task ServerStream(Req req, IServerStreamWriter<Res> rs, ServerCallContext ctx) {
  for (int i=0;i<5;i++){
    await rs.WriteAsync(new Res { Msg = $"{req.Msg} #{i}" });
    await Task.Delay(500);
  }
}
```

### Client Streaming

#### Client

```csharp
using var call = client.ClientStream();
for(int i=0;i<5;i++)
  await call.RequestStream.WriteAsync(new Req { Msg = $"C{i}" });
await call.RequestStream.CompleteAsync();
var r = await call;
Console.WriteLine(r.Msg);
```

#### Server

```csharp
public override async Task<Res> ClientStream(IAsyncStreamReader<Req> rs, ServerCallContext ctx) {
  var sb = new StringBuilder();
  await foreach (var r in rs.ReadAllAsync()) sb.Append(r.Msg);
  return new Res { Msg = sb.ToString() };
}
```

### Bi-Directional Streaming

#### Client

```csharp
using var call = client.BidiStream();
var readTask = Task.Run(async () => {
  await foreach(var r in call.ResponseStream.ReadAllAsync())
    Console.WriteLine("S: " + r.Msg);
});
foreach(var m in new[]{"A","B","C"}){
  await call.RequestStream.WriteAsync(new Req{Msg=m});
}
await call.RequestStream.CompleteAsync();
await readTask;
```

#### Server

```csharp
public override async Task BidiStream(IAsyncStreamReader<Req> rs, IServerStreamWriter<Res> ws, ServerCallContext ctx){
  await foreach(var r in rs.ReadAllAsync()){
    await ws.WriteAsync(new Res{Msg="Ack "+r.Msg});
  }
}
```
