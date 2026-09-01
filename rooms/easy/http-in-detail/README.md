# HTTP in Detail

**Platform:** TryHackMe · **Type:** Concept Guide (no personal run captured — see note below)
**Primary domain:** Web Fundamentals

> **Note:** This is a concept guide covering the material this TryHackMe room teaches, written from
> public documentation and reference material. It is not a personal walkthrough — no session evidence
> (screenshots, command output, room-specific answers) is included.

## Overview

The Hypertext Transfer Protocol (HTTP) is the application-layer protocol that underlies virtually all
web traffic: it defines how a client asks a server for a resource and how the server describes what it is
sending back. Modern HTTP is specified across a small family of RFCs published by the IETF's HTTPBIS
working group — [RFC 9110](https://www.rfc-editor.org/rfc/rfc9110) defines the semantics shared by every
HTTP version, [RFC 9112](https://www.rfc-editor.org/rfc/rfc9112) defines the HTTP/1.1 message syntax on
the wire, and companion RFCs cover caching, conditional requests, and range requests. Understanding HTTP
at the message level — not just "what a browser does" — is what makes it possible to reason precisely
about a huge share of web application vulnerabilities, since most of them are really deviations in how a
client, proxy, or server parses or trusts part of an HTTP message.

## Core Concepts

### Request structure

An HTTP request is composed of a **request line**, a set of **header fields**, an optional blank-line
separator, and an optional **body**. The request line names a **method**, a **request-target** (usually a
path plus query string), and the protocol version, e.g. `GET /search?q=dns HTTP/1.1`. Headers are
colon-separated `name: value` pairs that carry metadata — `Host` (mandatory in HTTP/1.1, identifying which
virtual host on the server should handle the request), `User-Agent`, `Accept`, `Content-Type`,
`Content-Length`, `Cookie`, and many others. The formal grammar for this is defined in
[RFC 9112 §3](https://www.rfc-editor.org/rfc/rfc9112).

### Methods

HTTP defines a fixed set of methods, each with defined semantics around safety and idempotency
([RFC 9110 §9](https://www.rfc-editor.org/rfc/rfc9110)):

| Method | Safe | Idempotent | Typical use |
|--------|------|------------|-------------|
| `GET` | Yes | Yes | Retrieve a representation of a resource |
| `HEAD` | Yes | Yes | Like GET, but headers only, no body |
| `POST` | No | No | Submit data to be processed (often creates a resource) |
| `PUT` | No | Yes | Replace a resource entirely with the request body |
| `DELETE` | No | Yes | Remove a resource |
| `PATCH` | No | No | Apply a partial modification to a resource |
| `OPTIONS` | Yes | Yes | Discover which methods/headers a resource supports (also used as a CORS preflight) |
| `TRACE` | Yes | Yes | Diagnostic loopback of the request as received |

**Safe** means the method is not expected to change server state (though servers are not literally
prevented from doing so); **idempotent** means issuing the same request multiple times has the same
effect as issuing it once. These properties matter for caching, retry logic, and for reasoning about
CSRF risk — a state-changing action exposed behind a "safe" GET request is itself a common vulnerability.

### Response structure and status code classes

A response mirrors the request's shape: a **status line** (protocol version, a three-digit status code,
and a human-readable reason phrase), headers, and an optional body. Status codes are grouped into five
classes ([RFC 9110 §15](https://www.rfc-editor.org/rfc/rfc9110)):

- **1xx Informational** — the request was received and processing continues (e.g. `100 Continue`).
- **2xx Success** — the request was successfully received, understood, and accepted (`200 OK`,
  `201 Created`, `204 No Content`).
- **3xx Redirection** — further action is needed to complete the request (`301 Moved Permanently`,
  `302 Found`, `304 Not Modified`).
- **4xx Client Error** — the request contains a problem the client should fix (`400 Bad Request`,
  `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `429 Too Many Requests`).
- **5xx Server Error** — the server failed to fulfil a valid request (`500 Internal Server Error`,
  `502 Bad Gateway`, `503 Service Unavailable`).

### Headers that matter for security

- **`Content-Security-Policy` (CSP)** — an allow-list mechanism that restricts which sources a page may
  load scripts, styles, images, and other resources from, sharply reducing the impact of injected script
  in an XSS scenario.
- **`Set-Cookie` flags** — `Secure` (cookie only sent over HTTPS), `HttpOnly` (cookie inaccessible to
  JavaScript, mitigating session theft via XSS), and `SameSite=Strict/Lax/None` (controls whether the
  cookie is sent on cross-site requests, a primary CSRF defense). These are formalized in
  [RFC 6265bis](https://datatracker.ietf.org/doc/html/draft-ietf-httpbis-rfc6265bis) building on the
  original cookie mechanism in [RFC 6265](https://www.rfc-editor.org/rfc/rfc6265).
- **CORS headers** (`Access-Control-Allow-Origin`, `Access-Control-Allow-Credentials`, etc.) — relax the
  browser's Same-Origin Policy in a controlled way, defined by the WHATWG
  [Fetch standard](https://fetch.spec.whatwg.org/#http-cors-protocol); a misconfigured wildcard
  (`Access-Control-Allow-Origin: *`) combined with credentialed requests is a common misconfiguration.
- **`Strict-Transport-Security` (HSTS)** — tells the browser to only ever contact this host over HTTPS
  for a specified duration, preventing protocol-downgrade and SSL-stripping attacks after the first visit.
- **`X-Content-Type-Options: nosniff`** — stops browsers from MIME-sniffing a response into an
  unexpected, more dangerous content type than the one declared.

### HTTP/1.1 vs HTTP/2 vs HTTP/3

HTTP/1.1, defined in RFC 9112, is a text-based protocol where, by default, one request must complete
before the next is sent on a connection unless **pipelining** or multiple parallel connections are used;
message framing relies on either `Content-Length` or `Transfer-Encoding: chunked`. **HTTP/2**
([RFC 9113](https://www.rfc-editor.org/rfc/rfc9113)) replaces this with a binary framing layer that
multiplexes many concurrent request/response exchanges over a single TCP connection, adds header
compression (HPACK), and allows the server to proactively push resources. **HTTP/3**
([RFC 9114](https://www.rfc-editor.org/rfc/rfc9114)) keeps HTTP/2's semantics but replaces the TCP+TLS
transport with **QUIC** ([RFC 9000](https://www.rfc-editor.org/rfc/rfc9000)), a UDP-based transport that
removes head-of-line blocking at the transport layer and folds the TLS handshake into connection
establishment. Notably, the dual framing mechanisms in HTTP/1.1 (`Content-Length` vs
`Transfer-Encoding: chunked`, and how they interact across proxies) are the direct root cause of the
HTTP request smuggling class of vulnerabilities.

## Why It Matters for Security

- **HTTP request smuggling** — when a front-end proxy and a back-end server disagree about where one
  request ends and the next begins (typically due to ambiguous or conflicting `Content-Length` and
  `Transfer-Encoding` headers), an attacker can smuggle a second, hidden request that the back-end
  processes as if it came from the next legitimate client, enabling request hijacking, cache poisoning,
  and authentication bypass.
- **Missing or weak security headers** — the absence of CSP, HSTS, `X-Frame-Options`, or correct cookie
  flags doesn't create a vulnerability by itself, but it removes a defense-in-depth layer that would
  otherwise contain the impact of an XSS, clickjacking, or session-hijacking flaw elsewhere in the
  application.
- **Session cookie misconfiguration** — a session cookie missing `HttpOnly` is readable by any injected
  script, turning a reflected or stored XSS into full session takeover; a cookie missing `Secure` can be
  captured over a downgraded or on-path HTTP connection.
- **Host header injection** — applications that trust the `Host` header to build absolute URLs (password
  reset links, cache keys) without validating it against a known-good list can be manipulated into
  generating links that point at attacker-controlled infrastructure.
- **Verb tampering / method confusion** — servers or middleware that apply access control logic only to
  specific methods (e.g. blocking `POST` but not `PUT` or a non-standard verb) can be bypassed by using an
  unexpected but still-accepted method.

## Common Pitfalls / Misconfigurations

- **Wildcard CORS with credentials** — combining `Access-Control-Allow-Origin: *` with
  `Access-Control-Allow-Credentials: true` is invalid per the Fetch spec and browsers reject it, but
  looser variants (reflecting the request's `Origin` header verbatim) achieve the same dangerous effect
  and are common in the wild.
- **Relying on client-side redirects or status codes for authorization** — a `302` redirect to a login
  page is not access control; the original response body may already have been sent or may still be
  retrievable by a client that ignores the redirect.
- **Inconsistent framing between proxy and origin server** — mixing `Content-Length` and
  `Transfer-Encoding` handling across a reverse proxy chain, or trusting an ambiguous combination of both
  headers, is the single most common enabler of request smuggling.
- **Overlong-lived, non-rotating session tokens** — sessions without expiry, without rotation on
  privilege change (e.g. login), and without server-side invalidation on logout extend the blast radius of
  any token leak.
- **Verbose error responses** — `5xx` responses that leak stack traces, internal hostnames, or framework
  version banners hand reconnaissance information to an attacker for free.

## Related TryHackMe Rooms in This Series

- [DNS in Detail](../dns-in-detail/README.md) — covers the resolution step that happens immediately
  before any HTTP connection can be opened.
- [How Websites Work](../how-websites-work/README.md) — places the HTTP request/response exchange as
  Step 3 in the broader end-to-end pipeline of loading a web page.
- [Putting it all together](../putting-it-all-together/README.md) — ties DNS, transport, and HTTP
  together in a single end-to-end narrative.

## References

- [RFC 9110 — HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110)
- [RFC 9111 — HTTP Caching](https://www.rfc-editor.org/rfc/rfc9111)
- [RFC 9112 — HTTP/1.1](https://www.rfc-editor.org/rfc/rfc9112)
- [RFC 9113 — HTTP/2](https://www.rfc-editor.org/rfc/rfc9113)
- [RFC 9114 — HTTP/3](https://www.rfc-editor.org/rfc/rfc9114)
- [RFC 9000 — QUIC: A UDP-Based Multiplexed and Secure Transport](https://www.rfc-editor.org/rfc/rfc9000)
- [RFC 6265 — HTTP State Management Mechanism (Cookies)](https://www.rfc-editor.org/rfc/rfc6265)
- [WHATWG Fetch Standard — CORS protocol](https://fetch.spec.whatwg.org/#http-cors-protocol)
- [MDN — An overview of HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Overview)
- [MDN — HTTP response status codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status)
- [OWASP — HTTP Request Smuggling](https://owasp.org/www-community/attacks/HTTP_Request_Smuggling)
- [PortSwigger — HTTP request smuggling](https://portswigger.net/web-security/request-smuggling)
