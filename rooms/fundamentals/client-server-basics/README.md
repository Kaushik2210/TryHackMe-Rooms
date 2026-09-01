# Client-Server Basics

**Platform:** TryHackMe · **Type:** Concept Guide (no personal run captured — see note below)
**Primary domain:** Computing Fundamentals

> **Note:** This is a concept guide covering the material this TryHackMe room teaches, written from
> public documentation and reference material. It is not a personal walkthrough — no session evidence
> (screenshots, command output, room-specific answers) is included.

## Overview

Almost every interaction on the internet — loading a web page, sending an email, pulling a file from a remote server, or connecting to a TryHackMe attack box — follows the same basic pattern: one machine asks for something, and another machine provides it. This pattern is called the **client-server model**, and it is the architectural backbone of the modern internet. Understanding it is a prerequisite for understanding networking, web application security, and most attack techniques, because nearly every vulnerability class (injection, authentication bypass, server-side request forgery, and so on) is really a question of what a server does when a client sends it unexpected input.

## Core Concepts

### The request-response model

In the client-server model, a **client** is a program or device that initiates a connection and requests a resource or service, and a **server** is a program or device that listens for incoming connections and fulfils those requests. Communication proceeds as a **request-response cycle**: the client sends a request over a network, the server processes it and returns a response, and the connection may then be closed or reused for further requests. A web browser (client) requesting a page from a web server is the canonical example, but the same pattern underlies DNS lookups (a resolver client queries a DNS server), file transfers (an FTP/SFTP client talks to a file server), and remote administration (an SSH client connects to an SSH server/daemon).

Critically, the client and server are logical roles, not fixed hardware categories. A single physical machine can run both a client program and a server daemon simultaneously (for example, a laptop running a local web browser as a client while also running a local development web server), and a machine that is a server in one exchange can act as a client in another (a web server acting as a client when it queries a backend database server).

### Ports, sockets, and addressing

For a client to reach the right service on a server, it needs two pieces of information: the server's **IP address** (which machine) and a **port number** (which service or process on that machine). Servers "listen" on well-known ports for particular protocols — port 80 for unencrypted HTTP, port 443 for HTTPS, port 22 for SSH, port 53 for DNS — as registered by IANA in the Service Name and Transport Protocol Port Number Registry. The combination of an IP address, a port number, and a transport protocol (TCP or UDP) forms a **socket**, which the operating system uses to track an individual connection or communication channel.

### Client-server vs. peer-to-peer

The client-server model is often contrasted with the **peer-to-peer (P2P)** model. In P2P architectures, every participating node (a "peer") can act as both client and requester and as server and provider, without a strict distinction of roles and without necessarily relying on a single central machine to coordinate the exchange — as seen in protocols like BitTorrent, where a downloading peer can simultaneously upload pieces of the same file to other peers. Client-server architectures centralise control and data at the server, which simplifies access control, logging, and updates, but creates a single point of failure and a concentrated target for attackers. P2P architectures distribute load and remove the single point of failure, but make consistent access control and monitoring much harder, since there is no single choke point through which all traffic passes.

### Stateless vs. stateful protocols

A further distinction that matters for security is whether the server retains information about a client between requests. **HTTP is fundamentally stateless** — each request is processed independently, with no built-in memory of prior requests — which is why web applications need mechanisms like cookies, session tokens, or JSON Web Tokens layered on top of HTTP to implement login sessions and track user state. **Stateful protocols**, such as FTP's control connection or a raw TCP session, maintain context about the ongoing conversation for as long as the connection is open. Understanding which model a given protocol uses is directly relevant to session-hijacking and authentication-bypass vulnerabilities, most of which exploit weaknesses in how state is reconstructed on a fundamentally stateless transport.

### A worked example: an HTTP request-response exchange

A minimal request a client sends to a web server looks like this:

```
GET /index.html HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0
```

The server's response might be:

```
HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 138

<html>...</html>
```

The client opened a TCP connection to `example.com` on port 443 (or 80), sent a `GET` request naming the resource it wants, and the server responded with a status line (`200 OK`), headers describing the response, and the requested content. Every subsequent page load repeats this cycle — and every web application security test starts by understanding exactly this exchange, since it is the surface every client-facing attack (parameter tampering, header injection, response manipulation) operates on.

## Why It Matters for Security

The client-server model defines the attack surface for the overwhelming majority of network-based attacks:

- **Servers are the higher-value target** because they typically hold data, run with elevated privileges, and are reachable by many clients — which is why reconnaissance and enumeration (port scanning, service fingerprinting) focus on discovering what services a server exposes and on which ports.
- **Trusting client-supplied data is a root cause of many vulnerabilities.** A server that assumes a client will only ever send well-formed, expected input is vulnerable to injection attacks (SQL injection, command injection), because a malicious client can send anything it wants — validation and sanitisation must happen server-side.
- **Server-Side Request Forgery (SSRF)** specifically abuses the fact that a server can itself act as a client: if an attacker can influence which URL or host a server-side process connects to, they can trick the server into acting as a proxy against internal, otherwise-unreachable systems.
- **Man-in-the-middle attacks** target the channel between client and server, which is why encrypting the transport (TLS/HTTPS) and authenticating both ends matters — an attacker positioned on the network path can otherwise intercept or alter the request-response exchange.

## Common Pitfalls / Misconfigurations

- **Exposing administrative or backend services directly to the internet** — services meant to be reached only by internal clients (databases, management interfaces) are frequently left listening on public-facing IPs due to misconfigured firewall rules or cloud security groups.
- **Assuming client-side validation is sufficient** — JavaScript form validation in a browser is a usability feature, not a security control, since an attacker can bypass the client entirely and send crafted requests directly to the server (with tools like Burp Suite or `curl`).
- **Misunderstanding statelessness** — developers sometimes store sensitive session data insecurely (in a client-readable cookie, for instance) because they underestimate how easily HTTP's statelessness pushes state-management decisions into application code, where mistakes are common.
- **Leaving default or well-known ports exposed with default credentials** — because ports are standardised and predictable, a server that responds on a known port with default or weak credentials is trivially discoverable by automated scanning.
- **Confusing the client-server model with trust** — a server should never assume a request came from a legitimate client application just because it arrived on the expected port using the expected protocol; the protocol only defines format, not identity or intent.

## Related TryHackMe Rooms in This Series

- [Inside a Computer System](../inside-a-computer-system/README.md) — covers the hardware a client or server runs on.
- [Computer Types](../computer-types/README.md) — servers, desktops, and mobile devices as distinct device categories that commonly take on client or server roles.
- [Data Encoding](../../easy/data-encoding/README.md) — the encodings used to represent data as it travels between client and server.
- See also [Virtualisation Basics](../virtualisation-basics/README.md) for how servers are commonly deployed as virtual machines.

## References

- IETF, RFC 9110, "HTTP Semantics" — https://www.rfc-editor.org/rfc/rfc9110.html
- IETF, RFC 793, "Transmission Control Protocol" — https://www.rfc-editor.org/rfc/rfc793.html
- IANA, "Service Name and Transport Protocol Port Number Registry" — https://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xhtml
- Mozilla Developer Network, "How does the Internet work?" — https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Understanding_networking_infra/How_does_the_Internet_work
- Mozilla Developer Network, "An overview of HTTP" — https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview
- OWASP, "Server Side Request Forgery Prevention Cheat Sheet" — https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html
- Cloudflare Learning Center, "What is the client-server model?" — https://www.cloudflare.com/learning/network-layer/what-is-the-client-server-model/
