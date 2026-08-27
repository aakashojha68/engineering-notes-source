# Networking Foundations & DNS

## Concept
Every web request travels down a stack of layers, each responsible for one
job — from "what does this data mean" (HTTP) down to "how do bits physically
move" (network hardware). DNS sits at the very start of this journey,
translating a human-readable domain into the IP address that actually
routes traffic.

## Why it matters
Interviewers use this as a warm-up to check you understand what each layer
is *actually* responsible for — not memorized trivia, but "if TCP already
guarantees reliable delivery, why do we need HTTP on top of it?" This
foundation is what makes TCP, TLS, REST, and CORS all click into place later.

## How it works

**The full request journey:**

```
User
  │
  ▼
Browser
  │
  ▼
DNS Lookup ──────► domain → IP address
  │
  ▼
TCP Connection ──► reliable byte transport established
  │
  ▼
TLS Handshake ───► encryption negotiated (HTTPS only)
  │
  ▼
HTTP Request ────► actual application data sent
  │
  ▼
Server
  │
  ▼
HTTP Response
  │
  ▼
Browser
```

**The layer stack — each layer only knows its own job:**

```
Application  (REST / GraphQL / gRPC)   "what does this data MEAN?"
     ↓
HTTP                                    "what's the request/response format?"
     ↓
TLS (HTTPS)                             "how do we encrypt this?"
     ↓
TCP                                     "how do we reliably deliver bytes?"
     ↓
IP                                      "how do we address/route?"
     ↓
Network (Wi-Fi / Ethernet / Fiber)      "how do bits physically move?"
```

Each layer is deliberately ignorant of the layers above it — **TCP doesn't
know what "GET" or "JSON" means**, it just moves bytes reliably. This
separation of concerns is why you can swap HTTP for GraphQL over the same
TCP/TLS stack without changing anything below it.

**DNS — domain to IP:**

```
api.example.com
      │
      ▼
     DNS
      │
      ▼
  203.x.x.x
```

**Public vs Private IP — a typical real path:**

```
Laptop
192.168.1.10 (Private)
      │
      ▼
   Router
49.x.x.x (Public)
      │
      ▼
  Internet
      │
      ▼
Load Balancer / Public Server
203.x.x.x (Public)
      │
      ▼
  Backend
10.0.0.x (Private)
```

- **Public IP** — globally routable, reachable directly on the internet.
- **Private IP** — only valid inside a local/internal network
  (`192.168.x.x`, `10.x.x.x`, `172.16.x.x`–`172.31.x.x`); the internet
  doesn't know how to route to these directly.

## Common interview questions
- Walk me through what happens when you type a URL into the browser and hit enter.
- What does each layer (HTTP, TLS, TCP, IP) actually own — where does one layer's responsibility end?
- Why does a backend server typically have a private IP even though it's serving public traffic?
- Why can two different laptops on two different home networks both have the IP `192.168.1.10`?
  (Private IP ranges aren't globally unique — they're only unique within their own local network.)

!!! tip "Gotchas / follow-ups"
    - "What happens when you type a URL" is one of the most common opening
      system design questions — being able to narrate this cleanly, layer by
      layer, is a strong first impression.
    - DNS resolution itself has its own mini-hierarchy (root → TLD → authoritative
      nameserver) — worth a one-line mention if asked to go deeper, though rarely required at this level.
    - A backend behind a load balancer having a private IP is *normal and
      intentional* — it forces all traffic through the load balancer/gateway,
      which is a security and traffic-management chokepoint.

## Personal example
_(Add a real case — e.g. debugging a "why can't my local app reach the
backend" issue that turned out to be a private-IP/VPN routing problem.)_

## Related
- [TCP](tcp.md)
- [TLS / HTTPS](tls-https.md)
