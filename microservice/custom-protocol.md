---
title: Custom Protocol
---

## 1) Goals & use cases (scope)

**Decide:** primary goal (file transfer, streaming, RPC, telemetry, IoT
telemetry, messaging), expected scale (1k/sec vs millions/sec), typical network
environments (LAN, Internet, cellular), and latency/throughput targets.

**Why:** every other decision flows from the core use case.

**Trade-offs:** high throughput vs low latency; simplicity vs feature richness.

---

## 2) Host model: connection-oriented vs connectionless

**Decide:** TCP-style sessions (stateful) or UDP-style datagrams (stateless).

**Why:** affects reliability, ordering, NAT traversal, and state management.

**Trade-offs:** connectionless = simpler, lower overhead; connection-oriented =
reliability and congestion control built-in.

---

## 3) Reliability, ordering, duplication semantics

**Decide:** must the protocol guarantee delivery, ordering, or exactly-once
semantics? Support optional modes?

**Why:** directly influences framing, ACK scheme, retransmission, sequence
numbers.

**Trade-offs:** stronger guarantees add complexity and state.

---

## 4) Transport layering & reuse

**Decide:** will you build on an existing transport (TCP, UDP, QUIC) or
implement transport features yourself?

**Why:** reusing transports (e.g. QUIC) gives you proven congestion control,
encryption, and NAT friendliness. Implementing from scratch means more control
but more work and risk.

**Trade-offs:** time-to-market vs control and customization.

---

## 5) Wire format & serialization

**Decide:** binary vs text, endianness, alignment, minimal vs self-describing
(e.g. Protobuf/CBOR vs JSON/XML).

**Why:** affects performance, debugging, extensibility, and size.

**Trade-offs:** binary = efficient but harder to debug; text = human readable
but verbose.

---

## 6) Framing & message boundaries

**Decide:** how messages are delimited (length prefix, sentinel bytes, TLV,
chunking).

**Why:** ensures correct parsing over streams and supports partial reads/writes.

**Trade-offs:** length prefix is simple; sentinel may conflict with payload.

---

## 7) Versioning & extensibility

**Decide:** header version fields, backward compatibility strategy, optional
TLVs, negotiation.

**Why:** protocols evolve; design for future without breaking existing
deployments.

**Trade-offs:** strict versioning is safer but less flexible vs extensible
optional fields requiring careful parsing.

---

## 8) Negotiation & handshake

**Decide:** negotiation steps on connection setup (capabilities, features, MTU,
compression, auth). Keep it minimal and secure.

**Why:** avoids costly mid-session surprises and supports heterogeneous clients.

**Trade-offs:** longer handshake adds latency but improves clarity and security.

---

## 9) Security: encryption, authentication, integrity

**Decide:** mandatory encryption (TLS/DTLS/QUIC) vs optional, authentication
modes (mutual TLS, token, API key), integrity checks, replay protection.

**Why:** security is often critical and should be chosen early.

**Trade-offs:** mandatory encryption increases complexity but is expected on the
modern Internet.

---

## 10) Threat model & privacy

**Decide:** likely attackers (MITM, replay, node compromise), data sensitivity,
logging/privacy policies (PII handling).

**Why:** design choices (e.g., ephemeral keys, PFS, minimal logging) come from
the threat model.

**Trade-offs:** stronger privacy can add operational complexity.

---

## 11) Error handling & diagnostics

**Decide:** canonical error codes, human-readable messages, error severity, and
recovery procedures. Decide what errors are fatal.

**Why:** operators and implementers need clear, debuggable failures.

**Trade-offs:** many error types increase clarity but complicate
implementations.

---

## 12) Flow control & congestion control

**Decide:** per-connection flow control, windowing, backpressure, and whether to
rely on lower-layer congestion control.

**Why:** prevents sender overwhelm and network collapse.

**Trade-offs:** implementing your own congestion control is complex; reusing
TCP/QUIC simplifies this.

---

## 13) Acknowledgement / retransmission strategy

**Decide:** ACK granularity (cumulative vs selective), timeouts,
retransmit/backoff algorithm, NACKs, FEC, or ARQ strategies.

**Why:** defines reliability, latency, and network behavior on packet loss.

**Trade-offs:** SACK/Selective ACKs are efficient but more complex.

---

## 14) Fragmentation & MTU handling

**Decide:** how to handle large messages: fragmentation at protocol level or
depend on IP fragmentation? Implement reassembly, maximum payload limits.

**Why:** IP fragmentation is unreliable; better to implement your own safe
fragmentation.

**Trade-offs:** application-layer fragmentation adds complexity but avoids path
MTU issues.

---

## 15) Multiplexing & streams

**Decide:** single stream per connection or multiplex multiple streams (like
HTTP/2 or QUIC).

**Why:** prevents head-of-line blocking and supports concurrent logical
channels.

**Trade-offs:** multiplexing requires stream prioritization and isolation logic.

---

## 16) Session & state management

**Decide:** how much server state to keep per client, session lifetime, session
resumption. Stateless vs stateful design.

**Why:** affects scalability, failover, and resource usage.

**Trade-offs:** stateless scales better; stateful supports richer semantics.

---

## 17) Addressing, routing, and identifiers

**Decide:** how endpoints are identified (IP\:port, GUID, URIs), support for NAT
traversal and proxies, and routing needs for multicast.

**Why:** determines how to locate and authenticate peers.

**Trade-offs:** using global identifiers simplifies discovery but raises privacy
and security concerns.

---

## 18) Discovery & bootstrap

**Decide:** how clients find servers: static config, DNS SRV, multicast
discovery, or a central registry.

**Why:** essential for deployment and scaling.

**Trade-offs:** centralized registries are powerful but single point of failure.

---

## 19) QoS, priority & traffic classes

**Decide:** support for prioritization, scheduling, or marking (DSCP), bandwidth
reservation for real-time traffic.

**Why:** controls service quality for mixed workloads (control vs bulk data).

**Trade-offs:** QoS improves important flows but complicates scheduling.

---

## 20) Multicast / Broadcast support

**Decide:** will protocol support multicast/broadcast; design message semantics
for those modes.

**Why:** important for live video, discovery, or telemetry.

**Trade-offs:** multicast can be efficient but has deployment and security
limitations.

---

## 21) Performance targets & benchmarks

**Decide:** latency P50/P95 targets, throughput goals, CPU/memory budget per
connection. Plan benchmark methodology.

**Why:** guides implementation choices and optimization.

**Trade-offs:** micro-optimizations may hurt readability or cross-platform
portability.

---

## 22) Resource limits & abuse mitigation

**Decide:** max concurrent connections, rate limits, quotas, authentication
throttling, and blacklisting.

**Why:** prevents DoS and resource exhaustion.

**Trade-offs:** aggressive limits can block legitimate clients.

---

## 23) Observability & metrics

**Decide:** what metrics to expose (latency, retransmits, bytes, active
sessions), logging format, tracing/span propagation.

**Why:** required for debugging and running at scale.

**Trade-offs:** verbose logging aids debugging but may expose sensitive data and
increase costs.

---

## 24) Conformance tests & reference implementation

**Decide:** produce a conformance suite, test vectors, fuzz harnesses, and at
least one reference implementation.

**Why:** interoperability depends on clear, testable specs.

**Trade-offs:** more upfront work but huge downstream savings.

---

## 25) Formal specification style & documentation

**Decide:** prose spec vs RFC-style with ABNF, message diagrams, state machines,
and examples. Include test vectors.

**Why:** clarity reduces misimplementations.

**Trade-offs:** formal specs are time-consuming but essential for adoption.

---

## 26) Backwards compatibility & upgrade strategy

**Decide:** how to roll out new versions, deprecate features, and negotiate
versions at runtime.

**Why:** makes upgrades safe in production.

**Trade-offs:** aggressive changes can fragment the ecosystem.

---

## 27) Licensing, governance & standardization

**Decide:** open standard vs proprietary, contribution model, and who controls
changes. Consider submitting to IETF etc.

**Why:** impacts adoption and ecosystem development.

**Trade-offs:** open standards aid adoption; proprietary control enables faster
pivots.

---

## 28) SDKs, reference clients & language bindings

**Decide:** which languages/platforms to support first, idiomatic APIs, and OS
integration.

**Why:** developer experience determines adoption.

**Trade-offs:** prioritizing many platforms slows initial delivery.

---

# Implementation artifacts you should produce early

- **Protocol overview** (one-page): goals, actors, state diagram.
- **Wire format spec**: header layout, TLVs, byte order, examples.
- **State machines** (connection setup, teardown, error states).
- **Security section**: handshake, key lifecycle, revocation.
- **Test vectors & conformance suite**: golden byte sequences, boundary cases.
- **Reference implementation**: minimal, well-tested server and client.
- **Observability plan**: metrics, logs, tracing ids.

---

# Common interview questions (and short model answers)

## _Why might you choose QUIC over TCP as a transport?_

QUIC provides multiplexed streams without head-of-line blocking, built-in TLS
1.3 for encryption and faster connection establishment (0-RTT in some cases),
and user-space deployability allowing faster iteration. Trade-off is more
implementation complexity and some middlebox incompatibilities.

## _How would you handle NAT traversal for peers behind home routers?_

Use hole punching (STUN) for UDP, TURN relay fallback, or rely on a central
rendezvous/relay server. Also consider using well-known ports or WebRTC/QUIC
which have NAT-friendly behaviors.

## _When would you include sequence numbers vs timestamps?_

Sequence numbers are necessary for ordering, deduplication, and retransmit.
Timestamps help with RTT measurement, jitter buffering, and time synchronization
tasks. Use both where appropriate.

## _How to design for backward compatibility?_

Use explicit version fields, optional TLVs, and capability negotiation. Avoid
breaking wire formats: add fields rather than change meanings.

## _How to mitigate replay attacks?_

Use nonces, sequence numbers with sliding windows, short-lived tokens, and
authenticated encryption with replay protection (e.g., AEAD + per-message IVs
derived from counters).

## _What metrics would you expose for operators?_

Active connections, P50/P95 latency, retransmit rate, throughput (bytes/sec),
error rates, handshake success/failure, auth failures, and resource usage per
node.
