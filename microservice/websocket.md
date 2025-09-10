---
title: WebSocket
---

## WebSocket: Protocol Overview, Pros, Cons, and Use Cases

### How WebSocket Works

**WebSocket** is a protocol standardized by the IETF in **RFC 6455**, enabling
full-duplex, bidirectional communication between a client and a server over a
single, persistent TCP connection. The protocol initiates with an HTTP handshake
where the client requests an upgrade to WebSocket, then both sides switch to the
`ws://` (unencrypted) or `wss://` (encrypted) protocol for ongoing
communication.

### WebSocket’s Advantages

The main strength of WebSocket lies in its efficiency and flexibility for **true
bidirectional, low-latency communication**. After the initial handshake, there’s
negligible protocol overhead because subsequent messages are framed with minimal
additional data—often as little as 2 bytes, compared to potentially
kilobyte-sized HTTP headers. This persistent link is ideal for workloads needing
**frequent updates** or instant event propagation, such as chat messaging,
multiplayer games, or collaborative tools.

### WebSocket’s Disadvantages

Nevertheless, WebSocket has several limitations. It’s not optimized for
**large-scale media streaming** (especially video/audio); in such cases,
**WebRTC** presents a more suitable alternative. While WebSocket offers reliable
real-time text/data streaming, its specification does not include built-in
reconnection or failover logic—developers must handle connection recovery,
heartbeats (PING/PONG), and error handling at the application level. Also,
WebSocket connections typically bypass intermediaries like HTTP caches and CDNs,
limiting the benefits of web infrastructure, and sometimes struggle with
enterprise proxies and firewalls if not correctly configured.

Security is another dimension: although WebSocket can be encrypted (wss),
developers must carefully manage **authentication and authorization**, as
persistent connections raise attack surface for session hijacking, cross-site
WebSocket hijacking, etc.. Finally, scaling WebSocket servers to handle vast
numbers of concurrent connections can be more complex than scaling stateless
HTTP servers, requiring thoughtful resource management and infrastructure
design.

### WebSocket Typical Use Cases

Thanks to its attributes, WebSocket is widely used for:

- **Real-time chat rooms and messaging platforms** (Slack, Discord, WhatsApp web
  clients)
- **Collaborative editing** (Google Docs, online whiteboards)
- **Online multiplayer games** with live event/position syncing
- **Financial trading platforms** and live ticker feeds
- **Collaborative drawing, live dashboards, and monitoring tools**
- **IoT device management** with instant notification and telemetry updates

---

## Server-Sent Events (SSE): Architecture, Pros, Cons, and Use Cases

### How SSE Works

**Server-Sent Events (SSE)** is an **HTTP-based protocol** that maintains a
persistent, one-way communication channel from the server to the browser.
Leveraging the `text/event-stream` MIME type, SSE enables a server to push
text-based updates over a continually open HTTP connection, which is managed by
the browser’s native **EventSource API**.

To establish an SSE connection, the client initiates a simple HTTP request; the
server responds with the appropriate headers, then streams data as needed. The
client passively listens for new messages, which arrive as pseudo-HTTP-chunked
events until the connection is closed or interrupted. Importantly, **the client
does not send data over the same connection**—if client-to-server messaging is
needed, a separate HTTP/AJAX channel (or another protocol) must be established.

SSE boasts **automatic reconnection**, meaning if a connection is lost (e.g.,
network failure or server restart), the browser attempts to transparently
re-establish it. The protocol also supports **custom event types, event IDs,
comments, and simple event formatting**.

### SSE’s Advantages

SSE is lauded for **simplicity**: it uses standard HTTP infrastructure, making
it inherently friendly to browsers, proxies, and load balancers, and is easy to
implement both in backend frameworks and modern frontend environments. Automatic
reconnection and message buffering provide reasonable resilience to temporary
network issues. SSE is well-suited to environments with many concurrent clients
requiring periodic updates (e.g., dashboards, live notifications), and works
seamlessly over both HTTP/1.1 and HTTP/2, the latter amplifying scalability via
multiplexing.

A distinguishing feature is its **compatibility with browser security and
authentication systems** (cookies, OAuth, etc.), since it operates as standard
HTTP. This attribute, along with its unidirectional nature, also limits certain
risks compared to bidirectional WebSocket sessions.

### SSE’s Disadvantages

The primary trade-off is its **unidirectionality**—SSE does not allow the
browser to send messages back over the open stream. For full duplex
(client-server interactivity), SSE must be paired with AJAX or other methods,
resulting in higher round-trip latency compared to WebSocket’s native two-way
channel. Additionally, SSE is **text-only** (no native binary frame support),
and is more susceptible to browser-imposed connection limits (notably in Chrome
and Firefox), which **limits the number of open concurrent SSE connections per
browser/domain**—problematic for apps opening many tabs or iframes.

While widely supported, **SSE lacks native support in some legacy browsers**
(notably Internet Explorer), and message size is constrained by HTTP
limitations. Scaling SSE may also stress server resources due to many long-lived
connections, similar to the challenges with long polling, although HTTP/2 helps
mitigate this with better multiplexing.

### SSE Typical Use Cases

SSE is ideal for:

- **Live data feeds**: news tickers, financial dashboards, sports scores
- **Monitoring boards**: flight status, operation centers, waiting room
  indicators
- **Social media streams**: Twitter feeds, Facebook notifications
- **Stock and cryptocurrency tickers**
- **Real-time analytics and activity streams**
- **Update notifications where only the server pushes data**

Its ease of implementation makes it a popular choice when interactivity
requirements are simple and one-way updates suffice.

---

## Long Polling: Mechanism, Pros, Cons, and Use Cases

### How Long Polling Works

**HTTP Long Polling** is a technique, rather than a formal protocol, that allows
web clients to receive near-real-time updates from servers over regular HTTP.
The **client periodically makes an HTTP request** (often a GET), but the server
keeps the connection **open and idle** until there is new data to send. Once a
message becomes available or a timeout occurs, the server sends a response and
closes the connection. The client then immediately issues a new request,
repeating the process indefinitely.

Unlike traditional polling, which sends requests at fixed intervals regardless
of data availability (potentially wasting resources), long polling reduces
unnecessary traffic by waiting until new information exists. **Long polling
emulates server push in otherwise one-way HTTP environments**, thus serving as a
bridge between classic HTTP and more advanced real-time protocols.

### Long Polling’s Advantages

Long polling requires **no changes to standard web infrastructure**, making it
universally compatible with all HTTP servers, proxies, firewalls, and browsers.
This makes it particularly valuable in **legacy or restricted environments** or
when supporting clients unable to use WebSocket or SSE due to institutional
policies (e.g., banking, government) or browser limitations.

Implementation is **straightforward** in almost any language or stack. Its
support for **bidirectionality** (as clients can send distinct HTTP requests)
allows some degree of two-way messaging—though with more overhead than
WebSocket. When properly managed, long polling can produce near-real-time effect
for moderate numbers of connections.

### Long Polling’s Disadvantages

**Scalability is the paramount limitation.** Each pending HTTP request in long
polling ties up server resources—connection pools, worker threads, and sometimes
memory—posing significant strain with thousands of concurrent users. As each
request may sit open for extended periods, this architecture puts a ceiling on
how many clients a typical web server can serve efficiently unless designed with
async/non-blocking I/O. The process also introduces latency spikes when
connections drop and need re-establishing, and results in higher round-trip
times and more prolonged reconnections than WebSockets or SSE.

Furthermore, **long polling is less efficient in terms of bandwidth**—each
exchange involves full HTTP headers, and more frequent connections mean
increased data transfer and socket churn compared to more persistent protocols.
Handling authentication, proxies, and session management adds redundancy and
subtle complexity. In general, long polling is considered a legacy fallback, fit
primarily for simpler, resource-tolerant deployments or maximal compatibility
scenarios.

### Long Polling Typical Use Cases

Long polling remains relevant for:

- **Older browser or firewall environments** lacking WebSocket/SSE support
- **Simple notification services** for moderate user volumes
- **Mobile apps** where connection constraints exist but polling is feasible
- **Incremental migration**: as fallback where modern protocols can’t be used

Prominent historical use cases include Facebook’s early chat implementation and
Gmail’s notifications, though virtually all have migrated to more advanced
protocols today due to scalability demands.

---

## WebRTC: Fundamentals, Pros, Cons, and Use Cases

### WebRTC Overview

**Web Real-Time Communication (WebRTC)** is a comprehensive, open-source project
and W3C standard that enables real-time, peer-to-peer audio, video, and
arbitrary data transmission directly between browsers and devices. It was
introduced to **eliminate the need for external plugins or proprietary
communication clients**, providing low-latency, encrypted communication
capabilities fully within the web platform.

WebRTC consists of intertwined APIs and protocols (not just one), including:

- **RTCPeerConnection** for media streams (audio/video)
- **RTCDataChannel** for arbitrary data
- **MediaStream** interfaces for capturing and sharing media inputs
- **ICE (Interactive Connectivity Establishment)**, **STUN/TURN servers** for
  NAT traversal, enabling connectivity even across home routers and firewalls

**Direct peer-to-peer connections** are established with help from signaling
servers (which negotiate connectivity but do not relay the media/data in most
use cases). The protocol suite automatically negotiates best codec, connection
path, and encryption (DTLS-SRTP), providing **secure, cross-platform,
high-fidelity communication**.

### WebRTC’s Advantages

WebRTC enables:

- **Seamless integration**: Audio, video, and data transfer in-browser with no
  plugins
- **Ultra-low latency**, critical for voice/video calls, live streaming,
  interactive experiences
- **End-to-end encryption** as default for streams
- **Direct P2P connectivity**, lowering server load and central infrastructure
  requirements
- **Cross-device compatibility**—works in most modern browsers, desktop, and
  mobile
- **Adaptive media handling** (dynamic bitrate adjustments, echo cancellation,
  noise suppression, packet loss concealment)
- **Rich use-case flexibility**, supporting data channels, file transfer, and
  streaming alongside media

Because of these attributes, WebRTC has become the backbone of major video
conferencing (Google Meet, Jitsi, Zoom’s web client, Discord calls), cloud
gaming, telehealth, and remote education solutions.

### WebRTC’s Disadvantages

WebRTC’s complexity is a double-edged sword. **Peer-to-peer connections are
difficult to scale** for large conference calls or multi-user sessions; unless
using selective forwarding units (SFU) or multipoint control units (MCU), mesh
architectures can quickly overload devices as participant count grows. Achieving
reliable NAT/firewall traversal can be challenging, sometimes requiring fallback
to relay (TURN) servers and introducing added bandwidth and cost.

**Browser support** is strong but not universal, with non-trivial differences in
implementation, requiring shims (like adapter.js) for cross-browser
compatibility. Additionally, security—while a major focus—demands careful,
ongoing attention from developers integrating WebRTC.

WebRTC may not be ideal for server-client streaming at huge scale (e.g.,
one-to-many live streams to millions), which may be better served by
CDN-accelerated streaming protocols like HLS or DASH.

### WebRTC Typical Use Cases

The strengths of WebRTC position it for:

- **Video conferencing and group video calls**
- **Voice calling and VoIP services**
- **Live streaming and broadcasting with ultra-low latency**
- **Online education (virtual classrooms, student engagement)**
- **Remote work (screen sharing, collaboration)**
- **Real-time gaming requiring synchronicity and minimal lag**
- **Augmented reality and telemedicine**
- **Secure, private communications between individuals or within organizations**

WebRTC adoption is surging across startups and enterprises, fueling new creative
and collaborative real-time applications worldwide.

---

### WebSocket Interview Questions

#### What is the WebSocket protocol and how does it differ from HTTP?

The WebSocket protocol, standardized in RFC 6455, enables full-duplex,
bidirectional communication between clients and servers over a single TCP
connection. While HTTP is based on a short-lived, request-response model
(one-way per request), WebSocket uses a persistent connection initiated with a
handshake, after which both server and client can send messages independently at
any time without additional handshakes or HTTP headers, resulting in lower
latency and overhead.

#### Describe a typical use case for WebSocket and why it’s preferred over alternatives

A classic use case is a live chat application. WebSocket is preferred because it
enables instant, bidirectional message exchange without polling, supporting high
concurrency and real-time user presence, which is inefficient or awkward to
implement with long polling or server-sent events.

#### What are key challenges or drawbacks to using WebSockets in a distributed system?

Scaling WebSocket connections is non-trivial—long-lived connections require
careful resource and session management. Load balancing, reconnection handling,
and communication with servers behind proxies/firewalls can be complex.
WebSocket does not leverage traditional HTTP caching or CDN acceleration, and it
lacks certain out-of-the-box features like automatic reconnection or fallback
mechanisms, which must be implemented in application logic.

#### How is binary data transmitted over WebSockets?

WebSocket supports binary data natively using frames—clients and servers can
send data as `ArrayBuffer` or `Blob` objects, reducing the need to encode binary
content as base64 or other textual representations.

#### How would you detect and handle loss of a WebSocket connection?

Connection loss can be detected with client-side event handlers (e.g.,
`onclose`, `onerror`). To ensure liveness, ping/pong heartbeats are implemented
periodically. Upon disconnection, the client should implement exponential
backoff reconnection strategies to restore the link, with expired session
handling as appropriate.

### Server-Sent Events (SSE) Interview Questions

#### What is SSE, and how does it differ from WebSocket?

Server-Sent Events (SSE) is a unidirectional, HTTP-based streaming protocol that
allows servers to push text-based updates to clients via the EventSource API.
The connection is initiated by the client but remains open for server-to-client
messages only. Unlike WebSockets, which are bidirectional, SSE cannot be used
for real-time client-to-server messaging over the same connection, nor does it
support binary encoding natively.

#### What built-in features make SSE resilient on unstable networks?

SSE clients automatically reconnect if the connection drops, reducing the need
for manual recovery logic. The protocol supports message retry fields, event IDs
for resuming missed messages, and transparent handling of many proxy/firewall
issues due to its HTTP basis.

#### What are key limitations of SSE compared to alternatives?

SSE is limited to text-based, unidirectional messaging; for client→server
communication, a separate HTTP channel is required. Browser support may be
incomplete on older browsers or restricted on mobile, and limits on the number
of concurrent connections may be an issue when many tabs/windows are opened to
the same domain.

#### Name a scenario where SSE is preferable over WebSocket

For live dashboards, price tickers, or server status boards where only
server→client updates are needed, SSE provides a simpler and more
resource-efficient implementation than WebSocket, with less developer effort and
infrastructure complexity.

### Long Polling Interview Questions

#### Describe the HTTP long polling process step by step.

First, the client sends an HTTP request (e.g., GET) to the server. The server
keeps the connection open, only responding when new data is available (or the
timeout expires). When the server replies (with data or empty), the client
immediately opens a new request. This cycle continues indefinitely, simulating a
push-like effect within HTTP’s request-response model.

#### What are the scalability challenges of long polling?

Each client connection allocates server resources for the duration of its open
request, consuming threads, connections, and potentially memory. With thousands
of clients, this exhausts server resources quickly. Asynchronous frameworks
mitigate this somewhat, but true scalability to millions of clients is much
harder compared to protocols designed for persistent, lightweight connections
like WebSocket.

#### When would you use long polling in a modern web application?

Long polling is best reserved for legacy environments, compatibility with older
browsers/firewalls, or as a temporary fallback. When neither SSE nor WebSocket
are available due to policy, infrastructure, or browser limits, long polling can
still deliver near-real-time updates with moderate workloads.

### WebRTC Interview Questions

#### Summarize the key architectural components of WebRTC.

WebRTC’s main architectural elements are the RTCPeerConnection (for establishing
and maintaining peer-to-peer connections), MediaStream (for capturing and
handling audio/video), RTCDataChannel (for arbitrary data exchange), and the use
of ICE, STUN, and TURN servers for NAT traversal and ensuring connectivity
across complex network topologies. The initial signaling occurs through external
channels (e.g., WebSocket, REST), while actual media and data flow directly
peer-to-peer when possible.

#### What is the role of STUN and TURN servers in WebRTC?

STUN servers reveal a client’s public-facing IP address and port, helping
clients establish direct connections through NAT devices. When direct
connectivity fails (e.g., behind symmetric NAT or firewall), TURN servers relay
the media/data, acting as intermediaries, though at the cost of additional
latency and server bandwidth.

#### What are primary strengths and limitations of WebRTC?

WebRTC delivers plugin-free, encrypted, low-latency voice, video, and data
communication between browsers. Strengths include ultra-low latency, native
browser support, and rich media handling. Limitations include the complexity of
peer-to-peer scaling, challenges of reliable NAT/firewall traversal, risk of
increased server costs with heavy TURN relay usage, and uneven browser support
on some older platforms.

#### Give three typical scenarios ideally suited to WebRTC.

- Video conferencing apps
- Live peer-to-peer gaming with voice communication
- Remote education with video, screen sharing, and collaborative drawing in real
  time

---

## Sample Implementation: C# SignalR (Backend) and Angular (Frontend) Chat Room

### C# (.NET) SignalR ChatHub Example

```csharp
// ChatHub.cs
using Microsoft.AspNetCore.SignalR;
using System.Threading.Tasks;

public class ChatHub : Hub
{
    public async Task SendMessage(string user, string message)
    {
        // Send to all connected clients
        await Clients.All.SendAsync("ReceiveMessage", user, message);
    }
}
```

**Explanation:** This class inherits from `Hub` and defines a method
`SendMessage` that clients call to send chat messages. All messages are
broadcast to every connected client via the `ReceiveMessage` event.

**Configure SignalR in Program.cs (.NET 6/7/8+):**

```csharp
using Microsoft.AspNetCore.Builder;
using Microsoft.Extensions.DependencyInjection;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddSignalR();
var app = builder.Build();

app.MapHub<ChatHub>("/chathub");
app.Run();
```

**Explanation:** Registers the SignalR service and maps the `ChatHub` endpoint
to `/chathub`—clients connect here to establish real-time messaging.

---

### Angular Client: SignalR Integration and Minimal Chat UI

```typescript
// signalr.service.ts
import { Injectable } from "@angular/core";
import * as signalR from "@microsoft/signalr";
import { Subject } from "rxjs";

@Injectable({ providedIn: "root" })
export class SignalRService {
  private hubConnection: signalR.HubConnection;
  private messageSubject = new Subject<{ user: string; message: string }>();
  messages$ = this.messageSubject.asObservable();

  startConnection() {
    this.hubConnection = new signalR.HubConnectionBuilder()
      .withUrl("https://localhost:5001/chathub")
      .build();

    this.hubConnection.start();

    this.hubConnection.on("ReceiveMessage", (user, message) => {
      this.messageSubject.next({ user, message });
    });
  }

  sendMessage(user: string, message: string) {
    this.hubConnection.invoke("SendMessage", user, message);
  }
}
```

```typescript
// chat.component.ts
import { Component, OnInit } from "@angular/core";
import { SignalRService } from "./signalr.service";

@Component({
  selector: "app-chat",
  template: `
    <input [(ngModel)]="user" placeholder="Your name" />
    <input [(ngModel)]="message" placeholder="Message" />
    <button (click)="send()">Send</button>
    <ul>
      <li *ngFor="let msg of messages">{{ msg.user }}: {{ msg.message }}</li>
    </ul>
  `,
})
export class ChatComponent implements OnInit {
  user = "";
  message = "";
  messages: { user: string; message: string }[] = [];

  constructor(private signalRService: SignalRService) {}

  ngOnInit() {
    this.signalRService.startConnection();
    this.signalRService.messages$.subscribe((msg) => this.messages.push(msg));
  }

  send() {
    if (this.user && this.message) {
      this.signalRService.sendMessage(this.user, this.message);
      this.message = "";
    }
  }
}
```

**Explanation:** The Angular service establishes a SignalR connection, exposes
an observable for messages, and provides a `sendMessage` method. The component
binds form fields to send messages and displays chat logs. This structure
adheres to Angular’s standards while being concise and testable.
