# TLS / HTTPS

## Concept
TLS (Transport Layer Security) encrypts the communication between client and
server. HTTPS is simply **HTTP running on top of a TLS-encrypted connection**
instead of plain TCP.

## Why it matters
This is where "how does the internet keep data private" actually gets
answered mechanically — and it clears up a genuinely common misconception:
that knowing a server's IP or its public key would somehow let you decrypt
its traffic. It doesn't, and knowing *why* is the interview-relevant part.

## How it works

```
TCP connection
      │
      ▼
TLS handshake ──► negotiate encryption, verify server identity, establish session keys
      │
      ▼
Encrypted communication
      │
      ▼
HTTP request/response (now flowing inside the encrypted tunnel)
```

The server holds a **certificate** and a **public/private key pair**.
Modern TLS uses a secure key exchange to establish a shared **session key**
for that specific connection — this is what actually encrypts the traffic,
not the server's long-term private key directly.

**The key misconception, cleared up:**

> Knowing the server's IP address does **not** let you derive its TLS session
> key. The public key is meant to be public — it's only useful for verifying
> identity and helping establish the session key, not for decrypting traffic on its own.

```
Analogy:
IP address  = house address   (public, anyone can find it)
Private key = house key       (only the owner has it)

Knowing the address doesn't give you the key.
```

## Common interview questions
- What's the difference between HTTP and HTTPS, mechanically?
- If someone knows a server's public IP, can they decrypt its traffic? Why not?
- What's the role of the server's certificate in the TLS handshake?
- Are JWT signing keys and TLS keys the same thing? (No — separate concerns:
  TLS keys secure the transport channel itself; JWT signing keys verify the
  authenticity of an application-level token traveling *through* that channel.)

!!! tip "Gotchas / follow-ups"
    - TLS handshake details (asymmetric key exchange to establish a symmetric
      session key, certificate chain validation) can go arbitrarily deep —
      for most interviews, understanding *why* it's secure (public key ≠
      private key, session keys are ephemeral) is enough; full handshake
      byte-level detail is rarely required outside security-focused roles.
    - A common follow-up: "why not just use asymmetric encryption for
      everything?" — Answer: it's computationally expensive; TLS uses
      asymmetric crypto only to safely establish a symmetric session key,
      then switches to fast symmetric encryption for the actual data.

## Personal example
_(Add a real case — e.g. debugging a "mixed content" or certificate error
when deploying a React app over HTTPS, or configuring HTTPS locally for testing.)_

## Related
- [Networking Foundations & DNS](networking-foundations.md)
- [TCP](tcp.md)
- [JWT & Cookies](cookies-and-auth.md)
