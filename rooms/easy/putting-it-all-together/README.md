# Putting it all together

**Platform:** TryHackMe · **Type:** Concept Guide (no personal run captured — see note below)
**Primary domain:** Web Fundamentals

> **Note:** This is a concept guide covering the material this TryHackMe room teaches, written from
> public documentation and reference material. It is not a personal walkthrough — no session evidence
> (screenshots, command output, room-specific answers) is included.

## Overview

Individually, DNS, TCP/IP, TLS, and HTTP are each well-documented protocols with their own RFCs. The
harder skill — and the one that actually matters for troubleshooting and for security assessment — is
tracing a single request through *all* of them as one continuous chain, and knowing which layer to
inspect when something goes wrong (a page won't load, a certificate warning appears, a request behaves
unexpectedly). This guide synthesizes the layers into one mental model and shows how to reason about a
request as it crosses each boundary, connecting the material covered separately in
[How Websites Work](../how-websites-work/README.md), [DNS in Detail](../dns-in-detail/README.md), and
[HTTP in Detail](../http-in-detail/README.md).

## Core Concepts

### Layering the stack

The internet protocol stack is commonly described using the four-layer TCP/IP model (link, internet,
transport, application) or the seven-layer OSI model. For web traffic, the layers that matter day to day
are:

| Layer | Protocol(s) | Responsibility |
|---|---|---|
| Application | DNS, HTTP, TLS (session) | Naming, request semantics, encryption |
| Transport | TCP (occasionally UDP/QUIC) | Reliable delivery, ordering, ports |
| Internet | IP (IPv4/IPv6) | Addressing, routing between networks |
| Link | Ethernet, Wi-Fi | Framing, delivery on the local segment |

A single page load touches every layer multiple times — once for the initial DNS query, once for the TCP
handshake to the web server, again for each subresource, and potentially again for third-party domains
(CDNs, analytics, fonts, ad networks) embedded in the page.

### Tracing one request end-to-end

Consider a browser requesting `https://shop.example/cart`:

1. **Naming (DNS).** The stub resolver in the OS asks a recursive resolver to resolve `shop.example`.
   The recursive resolver either answers from cache or walks the hierarchy (root → `.example` TLD →
   `shop.example` authoritative server) per [RFC 1035](https://www.rfc-editor.org/rfc/rfc1035). The
   answer is an A or AAAA record — an IPv4 or IPv6 address.
2. **Routing (IP).** That address is used to build IP packets. Each router between the client and the
   server makes an independent forwarding decision based on its routing table; the client has no
   guarantee about the path packets take.
3. **Transport (TCP).** A three-way handshake establishes a connection on port 443. TCP guarantees
   ordered, reliable delivery of the bytes that follow, retransmitting anything lost in transit.
4. **Encryption (TLS).** A TLS handshake negotiates a cipher suite and validates the server's
   certificate chain against a trusted root, then derives session keys. From this point every byte on
   the wire is encrypted; an on-path observer sees only the destination IP, TCP metadata, and (unless
   Encrypted Client Hello is used) the SNI hostname.
5. **Request (HTTP).** Inside the encrypted tunnel, the browser sends an HTTP request with a method,
   path, headers (`Host`, `Cookie`, `User-Agent`, etc.), and possibly a body.
6. **Application logic.** The server's application code (behind a reverse proxy, load balancer, or CDN
   edge node in most real deployments) processes the request — authenticating the session cookie,
   querying a database, rendering a template — and returns an HTTP response.
7. **Response handling.** The browser receives the response, and if it's HTML, restarts the
   render pipeline described in [How Websites Work](../how-websites-work/README.md): parse, build DOM
   and CSSOM, fetch subresources (repeating steps 1–6 for each new host), layout, and paint.

### Where infrastructure inserts itself

Real deployments rarely look like "browser talks directly to origin server." Common intermediaries
include:

- **DNS-level load balancing / GeoDNS** — the DNS answer itself can vary by client location, returning
  different IPs to route users to the nearest data center.
- **CDNs (Content Delivery Networks)** — the IP returned by DNS is often a CDN edge node, not the origin;
  the CDN may serve cached content directly or proxy the request onward, terminating TLS itself and
  re-establishing a separate connection to the origin.
- **Reverse proxies and load balancers** — sit in front of application servers, terminating TLS,
  routing requests to the correct backend, and often adding headers like `X-Forwarded-For` to preserve
  the original client IP for the backend to see.
- **WAFs (Web Application Firewalls)** — inspect HTTP requests for known attack patterns before they
  reach the application, sitting logically between the client and the origin.

Each intermediary is itself a client-server pair layered inside the outer request, and each one is a
place where the chain can be misconfigured — for example, a CDN configured to cache a response that
should have been personalized per-user (a “cache deception” or unintended cache poisoning scenario).

## Why It Matters for Security

Thinking in layers is what makes triage tractable. A "site is down" report could mean a DNS failure, a
TCP-level firewall block, a TLS certificate expiry, an HTTP 5xx from the application, or a rendering
failure in the browser — and the fix, and the tooling used to diagnose it (`dig`/`nslookup`, `curl -v`,
browser DevTools' Network tab, `openssl s_client`), differs completely depending on which layer failed.
On the offensive side, the same layering explains why attacks chain: an attacker who can poison a DNS
cache (layer 1) can redirect a victim to a server they control, which can then present a fraudulent TLS
certificate or none at all (layer 2), to serve a phishing page that harvests credentials via a form
submitted over HTTP (layer 3). Each stage is a smaller, well-understood problem; the "attack" is the
composition of several individually small failures.

## Common Pitfalls / Misconfigurations

- **Assuming DNS resolves to the origin server.** Many "the site is unreachable but the origin is fine"
  incidents trace back to a CDN or proxy layer, not the application itself — always confirm what IP a
  hostname actually resolves to and who owns it before assuming the origin is broken.
- **Forgetting that intermediaries can desynchronize.** A front-end proxy and a back-end server
  disagreeing about where an HTTP request ends (Content-Length vs. Transfer-Encoding) is the root cause
  of HTTP request smuggling — a layering bug, not a single-protocol bug (see
  [HTTP in Detail](../http-in-detail/README.md)).
- **Trusting `X-Forwarded-For` blindly.** This header is set by whichever intermediary is directly
  adjacent to the backend, but it can be spoofed by anything upstream of the *first* trusted proxy if
  the chain of trust isn't enforced.
- **Debugging at the wrong layer.** Restarting an application server won't fix a DNS TTL issue, and
  clearing a browser cache won't fix an expired TLS certificate — matching the symptom to the correct
  layer saves significant troubleshooting time.

## Related TryHackMe Rooms in This Series

- [How Websites Work](../how-websites-work/README.md) — the per-layer breakdown this room's synthesis
  builds on.
- [DNS in Detail](../dns-in-detail/README.md) — deep dive on the naming layer referenced in step 1.
- [HTTP in Detail](../http-in-detail/README.md) — deep dive on the application-layer exchange referenced
  in steps 5–6.

## References

- [RFC 1035 — Domain Names: Implementation and Specification](https://www.rfc-editor.org/rfc/rfc1035)
- [RFC 9110 — HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110)
- [RFC 9293 — Transmission Control Protocol (TCP)](https://www.rfc-editor.org/rfc/rfc9293)
- [RFC 8446 — TLS 1.3](https://www.rfc-editor.org/rfc/rfc8446)
- [RFC 791 — Internet Protocol](https://www.rfc-editor.org/rfc/rfc791)
- [MDN — An overview of HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Overview)
- [MDN — Proxy servers and tunneling](https://developer.mozilla.org/en-US/docs/Web/HTTP/Proxy_servers_and_tunneling)
- [Cloudflare Learning Center — What happens when you type a URL into your browser?](https://www.cloudflare.com/learning/dns/what-is-dns/)
