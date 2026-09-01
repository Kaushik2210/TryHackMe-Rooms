# How Websites Work

**Platform:** TryHackMe · **Type:** Concept Guide (no personal run captured — see note below)
**Primary domain:** Web Fundamentals

> **Note:** This is a concept guide covering the material this TryHackMe room teaches, written from
> public documentation and reference material. It is not a personal walkthrough — no session evidence
> (screenshots, command output, room-specific answers) is included.

## Overview

Every time a browser loads a page, a short but intricate sequence of network and software events takes
place: a name has to be resolved to an address, a connection has to be established, a request has to be
formed and sent, a server has to generate a response, and the browser has to turn that response into
pixels on a screen. Understanding this pipeline end-to-end is foundational for web security work — most
web application vulnerabilities (and most web-layer attacks, from DNS spoofing to XSS) are really just
abuses of one stage of this pipeline behaving in an unexpected way. This guide walks through the pipeline
at a conceptual level, with pointers to the deeper protocol guides in this series.

## Core Concepts

### The client-server model

The web is built on a client-server architecture. A **client** (typically a browser, but also curl, a
mobile app, or a script) initiates a request; a **server** listens for connections and returns a
response. This is the foundation of HTTP as defined in [RFC 9110](https://www.rfc-editor.org/rfc/rfc9110)
(HTTP Semantics) — the client-server exchange is stateless by default, meaning each request is handled
independently unless state is deliberately layered on top (cookies, sessions, tokens).

### Step 1 — Turning a name into an address

A user rarely types an IP address; they type a domain name like `example.com`. Before any connection can
happen, that name must be resolved to an IP address via the Domain Name System (DNS). This involves:

1. Checking local caches (browser cache, OS resolver cache, `hosts` file).
2. Querying a configured **recursive resolver** (often the ISP's resolver, or a public one like
   1.1.1.1 or 8.8.8.8).
3. The recursive resolver walking the DNS hierarchy — root servers, then TLD servers, then the
   **authoritative name server** for the domain — until it gets an answer, which it caches and returns.

This process, its record types, and its wire format are covered in depth in
[DNS in Detail](../dns-in-detail/README.md). The relevant base specification is
[RFC 1035](https://www.rfc-editor.org/rfc/rfc1035).

### Step 2 — Establishing a transport connection

Once the browser has an IP address, it opens a TCP connection to the server, almost always on port 443
(HTTPS) or port 80 (HTTP). TCP's three-way handshake (SYN, SYN-ACK, ACK) establishes a reliable,
ordered, bidirectional byte stream, as defined in [RFC 9293](https://www.rfc-editor.org/rfc/rfc9293).
For HTTPS, a **TLS handshake** immediately follows the TCP handshake: the client and server negotiate a
protocol version and cipher suite, the server presents an X.509 certificate proving its identity, and
both sides derive a shared symmetric key used to encrypt everything that follows. TLS 1.3
([RFC 8446](https://www.rfc-editor.org/rfc/rfc8446)) reduced this to a single round trip in the common
case, compared to two round trips in TLS 1.2.

### Step 3 — The HTTP request/response exchange

With an encrypted (or plaintext) connection open, the browser sends an HTTP **request**: a method
(`GET`, `POST`, etc.), a path, headers, and an optional body. The server processes the request — which
might mean serving a static file, querying a database, calling other internal services, or running
application logic — and returns an HTTP **response**: a status line, headers, and a body (typically
HTML, JSON, or a binary asset). The full anatomy of this exchange, including headers that matter for
security, is covered in [HTTP in Detail](../http-in-detail/README.md), based on
[RFC 9110](https://www.rfc-editor.org/rfc/rfc9110) and [RFC 9112](https://www.rfc-editor.org/rfc/rfc9112).

### Step 4 — Parsing and rendering

Once the browser has the HTML response body, it builds a **DOM (Document Object Model)** tree by parsing
the markup, and a **CSSOM (CSS Object Model)** tree by parsing stylesheets. These are combined into a
**render tree**, which the browser lays out (computing the geometry of every visible box) and then paints
to the screen. While parsing HTML, the browser discovers additional resources referenced by the page —
CSS files, JavaScript files, images, fonts — and issues further requests for each, repeating steps 1–3
for each new host involved. JavaScript execution can further mutate the DOM after the initial paint,
which is why modern pages often render progressively rather than all at once. The W3C/WHATWG documents
this pipeline in the [HTML Living Standard](https://html.spec.whatwg.org/multipage/) and MDN summarizes
it in [Populating the page: how browsers work](https://developer.mozilla.org/en-US/docs/Web/Performance/Guides/How_browsers_work).

### Putting a timeline together

A simplified timeline for loading `https://example.com/`:

```
1. DNS lookup:        example.com -> 93.184.216.34         (UDP/53 or DoH/DoT)
2. TCP handshake:      client <-> 93.184.216.34:443          (SYN, SYN-ACK, ACK)
3. TLS handshake:      negotiate cipher, verify cert, derive keys
4. HTTP request:        GET / HTTP/1.1
                        Host: example.com
5. HTTP response:       HTTP/1.1 200 OK
                        Content-Type: text/html
                        <html>...</html>
6. Parse + render:      DOM + CSSOM -> render tree -> layout -> paint
7. Subresource fetch:   repeat 1-5 for CSS, JS, images, fonts
```

## Why It Matters for Security

Nearly every category of web attack maps to one stage of this pipeline:

- **DNS stage** — cache poisoning, spoofing, and subdomain takeover let an attacker redirect step 1 to
  an address they control, effectively controlling everything downstream.
- **Transport stage** — a missing or misconfigured TLS certificate, or an application that accepts
  plaintext HTTP, exposes the connection to on-path interception (a classic man-in-the-middle).
- **HTTP stage** — this is where most web application vulnerabilities live: injection flaws, broken
  authentication, insecure headers, request smuggling, and CORS misconfiguration all manipulate the
  request/response exchange itself.
- **Rendering stage** — Cross-Site Scripting (XSS) abuses the fact that the browser will execute
  script found in a response body as if the origin server intended it to run, turning a data
  channel into a code-execution channel.

Understanding which stage a vulnerability class lives in is what lets a security professional reason
about root cause instead of memorizing payloads.

## Common Pitfalls / Misconfigurations

- **Mixing HTTP and HTTPS resources** ("mixed content") — a page served over HTTPS that loads a script
  or stylesheet over plain HTTP undermines the confidentiality and integrity guarantees TLS was meant to
  provide, and modern browsers actively block or warn on this.
- **Trusting client-side rendering for security enforcement** — access control, input validation, and
  authorization decisions made only in JavaScript running in the browser can be bypassed entirely, since
  the client is untrusted by definition.
- **Ignoring caching layers** — DNS caches, browser caches, and CDN caches can all serve stale or
  poisoned data long after an underlying record or resource has changed, which is a common source of
  confusing behavior during incident response.
- **Skipping certificate validation in code** — scripts, mobile apps, or internal tools that disable TLS
  certificate checks (`verify=False`-style flags) silently remove the identity-proofing step of the TLS
  handshake, reopening the door to interception.

## Related TryHackMe Rooms in This Series

- [Putting it all together](../putting-it-all-together/README.md) — walks through the same pipeline
  again with a focus on tying the layers together end-to-end.
- [DNS in Detail](../dns-in-detail/README.md) — a deep dive into the resolution process summarized in
  Step 1 above.
- [HTTP in Detail](../http-in-detail/README.md) — a deep dive into the request/response exchange
  summarized in Step 3 above.

## References

- [RFC 1035 — Domain Names: Implementation and Specification](https://www.rfc-editor.org/rfc/rfc1035)
- [RFC 9110 — HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110)
- [RFC 9112 — HTTP/1.1](https://www.rfc-editor.org/rfc/rfc9112)
- [RFC 9293 — Transmission Control Protocol (TCP)](https://www.rfc-editor.org/rfc/rfc9293)
- [RFC 8446 — The Transport Layer Security (TLS) Protocol Version 1.3](https://www.rfc-editor.org/rfc/rfc8446)
- [MDN — How browsers work](https://developer.mozilla.org/en-US/docs/Web/Performance/Guides/How_browsers_work)
- [MDN — An overview of HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Overview)
- [WHATWG HTML Living Standard](https://html.spec.whatwg.org/multipage/)
