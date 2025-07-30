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
