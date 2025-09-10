---
title: HTTP
---

## HTTP Version Overview

### HTTP/1.1

- Text‑based protocol with one request per connection (or pipelined with limited
  support); keeps connections alive via persistent connections to reduce
  handshake overhead.

- Has severe **head‑of‑line (HOL) blocking** since if one request stalls,
  nothing else proceeds.

- Introduced **chunked transfer encoding**, pipelining (rarely used), and
  keep‑alive headers to reduce latency.

- Persistent connections are the default: Unless explicitly closed, a TCP
  connection can handle back-to-back HTTP requests and responses over time.

- Multiple sequential requests can share the same TCP connection. It does not
  support multiple parallel requests over a single TCP connection in the way
  HTTP/2 does.

- Browsers typically open multiple persistent connections in parallel (e.g. 4–8)
  to a server to improve performance, but each of those connections can serve
  multiple sequential requests.

### HTTP/2

- Binary framing, **multiplexing** multiple streams over a single TCP connection
  reduces latency.

- Header compression using **HPACK** cuts redundant header sizes.

- **Server push** supports sending resources before a client requests them (but
  often disabled due to inefficiency).

- Supports **stream prioritization** for critical resource loading.

- Still suffers HOL blocking at the TCP layer if packets get lost.

### HTTP/3

- Shifts transport from TCP to **QUIC over UDP**, adopting TLS 1.3 built-in.

- Supports **multiplexing without TCP-level HOL blocking**, since streams are
  independent.

- Achieves **0‑RTT** or **1‑RTT** (Round Trip Time) connection setup by merging
  transport and encryption handshakes.

  - 1‑RTT is the standard TLS 1.3 handshake requiring one round trip before data
    flows.
  - 0‑RTT allows the client to send application data immediately by resuming a
    previous session using a PSK, **without waiting for the handshake to
    complete**, eliminating that RTT.

- Mandatory encryption, connection migration (e.g. Wi‑Fi <-> cellular) via
  connection IDs, forward error correction, adaptive congestion control.

### QUIC (Quick UDP Internet Connections)

- UDP‑based transport protocol implemented in user‑space, not kernel space,
  enabling rapid evolution.

- Includes TLS handshake in the transport handshake to reduce latency.

- Stream‑level retransmissions isolate packet loss impact per stream (no HOL
  blocking).

- Provides connection migration using connection IDs, better handling NAT
  traversal, advanced congestion control, and optional forward‑error correction.

- Some studies show performance under high‑speed networks may lag behind
  TCP+HTTP/2 due to user‑space overhead and implementation differences.

### Summary

| Feature/Version       | HTTP/1.1             | HTTP/2                 | HTTP/3 (QUIC/HTTP‑over‑UDP)                |
| --------------------- | -------------------- | ---------------------- | ------------------------------------------ |
| Transport Protocol    | TCP                  | TCP                    | QUIC over UDP                              |
| Request multiplexing  | No (serial/pipeline) | Yes                    | Yes (independent streams)                  |
| Head‑of‑Line blocking | Yes                  | TCP-level HOL possible | No HOL blocking across streams             |
| Header compression    | No                   | HPACK                  | QPACK or HPACK-style                       |
| Server Push           | No                   | Yes (often disabled)   | Yes                                        |
| Connection setup      | TLS + TCP handshake  | TCP + TLS handshake    | Combined QUIC + TLS (0‑RTT or 1‑RTT)       |
| Encryption            | Optional             | Optional (TLS typical) | Mandatory (TLS 1.3 integrated)             |
| Migration support     | No                   | No                     | Yes (via connection IDs)                   |
| Typical latency       | Higher               | Lower than 1.1         | Lower than 2; especially on lossy/wireless |

## HPACK Header Compression

HTTP/2 improves bandwidth efficiency and reduces request latency via **HPACK**,
a new header compression mechanism tailored for HTTP headers. With
ever-increasing header redundancy (due to cookies, user-agents, etc.) and
repetitive transmission of the same fields across page requests, uncompressed
headers consume significant bandwidth.

HPACK achieves compact, secure header compression by:

- Using static and dynamic header tables
- Employing a canonical Huffman encoding for string literals
- Avoiding vulnerabilities like the CRIME attack that affected generic
  compression schemes such as DEFLATE.

### Static Table

The **static table** is a built-in, unchangeable lookup table of the 61 most
commonly-used HTTP header fields (and some common values) as determined during
the standardization process. Both the client and the server possess an identical
version of this static table, allowing them to encode and decode headers
efficiently via compact indices (rather than repeating full header
names/values).

**Examples of static table entries (abbreviated for illustration):**

| Index | Header Field    | Value         |
| ----- | --------------- | ------------- |
| 1     | :authority      |               |
| 2     | :method         | GET           |
| 3     | :method         | POST          |
| ...   | ...             | ...           |
| 8     | :path           | /index.html   |
| ...   | ...             | ...           |
| 33    | accept-encoding | gzip, deflate |
| ...   | ...             | ...           |

When a header matches a static entry, HPACK can merely transmit its index
number, often encoded in a single byte, yielding dramatic space
savings—especially for frequently repeated headers.

### Dynamic Table

The **dynamic table** is a per-connection, mutable cache that stores header
fields (name/value combinations) seen during the ongoing HTTP/2 session. Both
sender and receiver maintain synchronized versions of the dynamic table, which
grows and updates as new headers are transmitted.

**Dynamic Table Management:**

- The table is capped at an agreed-upon size (default: 4096 bytes; can be
  adjusted via SETTINGS frame).
- When new headers are indexed, they are appended.
- If the table exceeds its size, the oldest entries are evicted (FIFO).
- Both parties rely on updating the table only after the full header block is
  received (frame boundaries must be respected for table consistency).

**Efficiency:** Rather than transmitting repeated full headers like `cookie:
sessionid=...`, the sender refers to a table index if the value was recently
sent, shrinking the header payload size.

**Security:** The dynamic table's per-connection scope avoids cross-user
leakage—a key improvement over generic compression with shared state which
previously facilitated attacks like CRIME.

### Huffman Encoding in HPACK

In addition to table indexing, HPACK compresses header names and values using
**Huffman encoding**. HPACK defines a **static canonical Huffman code table**,
optimized for typical HTTP header character frequencies. Huffman encoding
replaces each character with a variable-length code: frequent characters use
fewer bits, rare characters use more.

**Key features:**

- **Canonical, static tree:** Both endpoints know the exact bit patterns for
  every byte value—no negotiation needed per connection.
- **Efficient encoding:** Reduces string literal length for typical header
  values.
- **Security:** Since the tree is static, no on-the-fly code negotiation or
  historical context is needed, minimizing risk.

**When is Huffman used?** HPACK allows either raw (ASCII/UTF-8) or
Huffman-encoded transmissions for header string literals. The sender chooses
case-by-case; Huffman encoding is most space-efficient for longer textual
values, but not always for short or symbol-heavy strings. A flag in the data
signals which encoding is used for each string.

#### Huffman Code Table (Summary)

HPACK's static Huffman code table maps all 256 possible byte values to bit
patterns from 5 to 30 bits long, with smaller codes assigned to more frequent
characters. This table is published in [RFC 7541, Appendix B] and is integrated
in all compliant HTTP/2 implementations.

#### Huffman Encoding Example

**Encoding the string "!$%&A" with HPACK’s Huffman table:**

Let's break down the process:

| Character | Huffman Code                                  | Bit Length |
| --------- | --------------------------------------------- | ---------- |
| `!`       | `1111111000`                                  | 10         |
| `$`       | `11111110001111111111001`                     | 23         |
| `%`       | `11111110001111111111001010101`               | 27         |
| `&`       | `1111111000111111111100101010111111000`       | 34         |
| `A`       | `1111111000111111111100101010111111000100001` | 41         |

- Each character is replaced by the corresponding bit sequence (from static
  HPACK table).
- The sequences are concatenated to form a bitstream.
- Padding or an end-of-string indicator (EOS) may be added to ensure the bit
  stream aligns to byte boundaries.

**Example in binary (concatenated):**

```
1111111000
11111110001111111111001
11111110001111111111001010101
1111111000111111111100101010111111000
1111111000111111111100101010111111000100001
[EOS or padding as needed]
```

## Interview Questions

### Explain HTTP/1.1, HTTP/2, HTTP/3, and QUIC in a nutshell

HTTP/1.1 is a text‑based protocol using TCP, supporting persistent connections
but with head‑of‑line blocking and no multiplexing. HTTP/2 introduced binary
framing, header compression (HPACK), multiplexing, server push, and
prioritization, but still runs over TCP, so packet loss triggers HOL blocking at
the connection level. HTTP/3 uses QUIC over UDP with built‑in TLS 1.3,
multiplexing without TCP‑level HOL blocking, faster handshake, connection
migration, and better congestion control. QUIC itself is the transport protocol
enabling these features.

### What is head‑of‑line blocking and how do HTTP/2 and HTTP/3 address it?

Head‑of‑line blocking occurs when one packet loss stalls the entire connection.
In HTTP/2, although multiple streams exist, they share a TCP connection: if a
packet is lost, all streams stall until retransmission. HTTP/3 mitigates this by
using QUIC over UDP: each stream is independently managed, so loss in one stream
doesn’t delay others.

### How does HTTP/3 enable faster connection setup compared to HTTP/2?

QUIC integrates TLS 1.3 into its transport handshake, combining encryption and
connection setup. That enables 0‑RTT (if cached credentials are used) or at most
1 RTT setups, significantly reducing latency versus TCP+TLS's multi‑step
handshake.

### What is server push? When might it be counterproductive?

Server push allows the server to send assets (e.g. CSS/JS) before the client
requests them. It can reduce latency, but often leads to wasted bandwidth if the
client already has the resource or doesn't need it. Overuse may slow down
meaningful resource fetching and is nowadays often disabled.

### Why does QUIC run in user-space and what are the trade-offs?

QUIC is implemented in user-space (on top of UDP) so it can evolve faster and
bypass kernel-level ossification. The trade-off is potential performance
overhead: implementations may vary in efficiency, and in high-speed networks
QUIC sometimes underperforms TCP due to processing overhead.

### What improvements does HTTP/3 bring for mobile network scenarios?

HTTP/3 enables connection migration (via connection IDs), meaning if a device
switches from Wi‑Fi to cellular IP, the same QUIC connection persists—avoiding
re‑handshake. It also handles packet loss better, reducing load disruptions
common in mobile environments.

### What compression protocols are used in HTTP/2 and HTTP/3?

HTTP/2 uses HPACK for header compression. HTTP/3 uses QPACK, a similar header
compression mechanism adapted for QUIC's transport and avoiding head-of-line
issues.

### How does chunked transfer encoding in HTTP/1.1 work?

Chunked encoding sends response data in chunks so the client can begin
processing before the entire payload is generated.

### What is Multiplexing?

**Multiplexing** in HTTP/2 is the ability to send multiple HTTP streams (logical
request/response pairs) concurrently over a single, persistent TCP connection.
Streams are split into _frames_, which can be interleaved freely during
transmission, letting browsers, proxies, and servers exchange multiple requests
and responses simultaneously without having to wait for any one to finish first.

- **Stream:** A bidirectional exchange of frames (typically for one
  request/response pair), identified by a unique Stream ID.
- **Frame:** The basic HTTP/2 protocol unit—a small encapsulated chunk of data,
  such as HEADERS, DATA, or PUSH_PROMISE.
- **Multiplexing:** The client/server breaks down each stream into frames, then
  interleaves these frames over the same TCP connection. Each frame includes its
  Stream ID, so reassembly is straightforward at the receiving end.

### What is Stream Prioritization?

Clients can assign priority or weight to streams, advising servers of the
relative importance (e.g., “load CSS before images”). Servers can then schedule
transmission to favor these preferences, helping to optimize content delivery
for fast visual paint and interactivity.

### What is Flow Control?

To prevent buffer overruns and resource exhaustion, HTTP/2 implements per-stream
and per-connection flow control mechanisms. Each endpoint can limit how much
unacknowledged data it is willing to receive, contributing to fairness and
stability in high-concurrency scenarios.

---

## HTTPS

HTTPS (HyperText Transfer Protocol Secure) is simply HTTP layered over TLS
(Transport Layer Security), formerly known as SSL (Secure Sockets Layer).

### How SSL/TLS Works: The Secure Connection Process

#### Certificate & Trust Establishment

- Servers generate a public/private key pair and create a **Certificate Signing
  Request (CSR)** with details like domain name, then submit to a **Certificate
  Authority (CA)**.

- The CA verifies identity and issues a certificate signed with its private key,
  establishing a **chain of trust** that browsers validate using trusted root
  certificates.

#### TLS Handshake

- ClientHello: Client sends supported TLS version, cipher suites, and a random
  nonce (client random).

- ServerHello: Server responds with chosen cipher suite, its own nonce, and
  sends its certificate.

- Key Exchange:

  - Pre‑TLS 1.3: Client encrypts a **PreMasterSecret** with server's public key;
    server decrypts with its private key.
  - TLS 1.3: Uses Diffie‑Hellman (usually Elliptic Curve) for forward secrecy.

- Change Cipher Spec & Finished: Both sides signal that subsequent messages are
  encrypted using the derived symmetric session key.

- Session Resumption (optional): Allows abbreviated handshake using stored
  session IDs or tickets for quicker reconnection.

#### Encrypted Communication & Closure

- After the handshake, HTTP messages are encrypted using the negotiated
  symmetric cipher and session key.

- Connection closes via normal TLS closure or timeout.

### mTLS (mutual TLS)

Extends TLS by requiring both client and server to present and verify
certificates, enabling bi-directional authentication and significantly raising
security against impersonators or unauthorized access. mTLS is essential in
zero-trust architectures, internal microservice communication, and
high-assurance environments (e.g., banking, IoT). Drawbacks include increased
device management complexity and certificate lifecycle overhead, but the
trade-off is much stronger protection, especially against man-in-the-middle
attacks and credential theft.

---

## OSI Model

The OSI (Open Systems Interconnection) model is a 7‑layer reference framework
created by ISO to standardize network communication functions and foster
interoperability.

| Layer (7 -> 1)      | Key Responsibilities                                                                           |
| ------------------- | ---------------------------------------------------------------------------------------------- |
| **7. Application**  | Interfaces apps with network protocols (e.g. HTTP, FTP, DNS, SMTP)                             |
| **6. Presentation** | Formats, encrypts, compresses data (e.g. TLS/SSL, MIME, JSON/XML translation)                  |
| **5. Session**      | Manages sessions, synchronization, full/half duplex communication, RPC, NetBIOS                |
| **4. Transport**    | Ensures end-to-end delivery: segmentation, flow/error control, reliability (TCP/UDP/SCTP)      |
| **3. Network**      | Logical addressing, routing packets across networks (IP, ICMP, OSPF, BGP)                      |
| **2. Data Link**    | Framing, MAC addressing, error detection & correction (CRC), flow control; sublayers LLC & MAC |
| **1. Physical**     | Transmits raw bits over physical media; handles cables, voltage, signal encoding, topology     |

### Interview Questions

#### What is the OSI model and what are its benefits?

The OSI model (Open Systems Interconnection) is a conceptual framework defining
7 layers of network communication. It enables modular design, standardization,
interoperability, and helps isolate and troubleshoot issues at specific layers.

#### Can you name the 7 layers and explain each briefly?

- Application: Interfaces with user apps (HTTP, DNS).
- Presentation: Data translation, encryption/compression.
- Session: Establishing, managing, and terminating sessions.
- Transport: Reliable, ordered, and connection multiplexing with TCP/UDP.
- Network: Routing and logical IP addressing.
- Data Link: Framing and error control on physical links.
- Physical: Physical transmission of raw bits over media.

#### What protocols/devices operate at each layer?

- **Application**: HTTP, FTP, SMTP, DNS, DHCP, SNMP
- **Presentation**: TLS/SSL, data formats (MIME, JSON)
- **Session**: RPC, NetBIOS, session-control services
- **Transport**: TCP, UDP, SCTP, QUIC (as similar)
- **Network**: IP, ICMP, IGMP, OSPF, BGP
- **Data Link**: ARP, MAC addressing, Ethernet, PPP, switches, bridges
- **Physical**: Ethernet cabling, fiber, Wi‑Fi radio signals, hubs, repeaters

#### What's encapsulation and decapsulation?

Encapsulation wraps data with headers/trailers as it moves down the layers: each
layer adds info specific to its function. Decapsulation removes these as packets
travel upward. This modular layering helps maintain separation of concerns in
communication.

#### What distinguishes TCP from UDP at the Transport layer?

- **TCP**: connection-oriented, reliable, ordered delivery, flow control,
  retransmission.
- **UDP**: connectionless, faster, no fault recovery, suited for streaming or
  DNS.

#### How do router and switch differ in OSI context?

- **Switch**: operates at **Data Link** (Layer 2), forwarding frames using MAC
  addresses.
- **Router**: operates at **Network** (Layer 3), forwarding IP packets between
  networks using routing tables.

#### How do CRC and error handling work at Data Link layer?

The Data Link layer adds a CRC checksum to each frame. The receiver recalculates
the CRC and compares it: if mismatched, a retransmission is requested. Flow
control ensures sender doesn't outpace receiver.
