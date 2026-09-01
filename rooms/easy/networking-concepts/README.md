# Networking Concepts

**Platform:** TryHackMe · **Type:** Concept Guide (no personal run captured — see note below)
**Primary domain:** Networking

> **Note:** This is a concept guide covering the material this TryHackMe room teaches, written from
> public documentation and reference material. It is not a personal walkthrough — no session evidence
> (screenshots, command output, room-specific answers) is included.

## Overview

Before diving into individual protocols, a security practitioner needs a mental model for how networked communication is organised: which conceptual "layer" a given piece of technology belongs to, how two processes on different machines actually locate and talk to each other, and how the abstract OSI reference model relates to the TCP/IP suite that the real internet runs on. TryHackMe's "Networking Concepts" room consolidates this — the OSI model, the TCP/IP model, ports, and sockets — into the working vocabulary used throughout every later networking and offensive-security room.

## Core Concepts

### The OSI model: seven conceptual layers

The **Open Systems Interconnection (OSI) model**, standardised by ISO/IEC 7498-1, describes networking as seven distinct layers, each providing services to the layer above it and consuming services from the layer below:

| Layer | Name | Examples |
|---|---|---|
| 7 | Application | HTTP, DNS, SMTP |
| 6 | Presentation | TLS/SSL encryption, character encoding, compression |
| 5 | Session | Session establishment/teardown (e.g., NetBIOS sessions, RPC) |
| 4 | Transport | TCP, UDP |
| 3 | Network | IP, ICMP, routing |
| 2 | Data Link | Ethernet, MAC addressing, switches |
| 1 | Physical | Cabling, radio signals, voltages |

The OSI model was never fully implemented as designed — it is used today as a *teaching and diagnostic* framework, not a literal description of any real protocol stack. Its main practical value is vocabulary: saying an attack or a device "operates at Layer 2" versus "Layer 3" versus "Layer 7" precisely communicates its scope (e.g., a Layer 2 attack like ARP spoofing cannot cross a router, but a Layer 7 attack like SQL injection is invisible to a firewall that only inspects IP and port).

### The TCP/IP model: four layers that actually run the internet

The **TCP/IP model** (also called the Internet Protocol Suite) is the model the internet is actually built on, and it predates OSI. It compresses OSI's seven layers into four:

| TCP/IP Layer | Roughly maps to OSI | Protocols |
|---|---|---|
| Application | 5, 6, 7 | HTTP, DNS, SSH, TLS |
| Transport | 4 | TCP, UDP |
| Internet | 3 | IP, ICMP |
| Link (Network Access) | 1, 2 | Ethernet, Wi-Fi, ARP |

TCP/IP's layering is documented across foundational RFCs, most notably RFC 791 (IP) and RFC 9293 (TCP, which consolidated and obsoleted the original RFC 793). Where OSI is a reference model designed by committee, TCP/IP is a description of protocols that were already in production use — which is why real-world troubleshooting and packet analysis tend to reference TCP/IP's four layers more often than OSI's seven, even though OSI terminology ("Layer 2," "Layer 7 firewall") remains the common shorthand.

### Ports: multiplexing a single IP address

A single host with one IP address can run many network services simultaneously (a web server, an SSH daemon, a mail server) because the transport layer adds a **port number** — a 16-bit value from 0 to 65535 — that identifies a specific process or service. IANA maintains the authoritative registry dividing the port space into:

- **Well-known ports (0–1023)** — reserved for standard services, e.g., HTTP (80), HTTPS (443), SSH (22), DNS (53), FTP (20/21).
- **Registered ports (1024–49151)** — assignable by IANA to specific applications on request.
- **Dynamic/private/ephemeral ports (49152–65535)** — used transiently, typically as the *source* port a client picks for an outbound connection.

A destination port tells a server which service the client wants; a source port lets the client's OS route the eventual reply back to the correct application instance.

### Sockets: the actual communication endpoint

A **socket** is the combination of an IP address, a port number, and a transport protocol (TCP or UDP) — conceptually, "how do I address this specific end of a specific conversation." The Berkeley sockets API (which most operating systems' networking libraries implement, directly or via a compatible interface) is the programming abstraction applications use to open, read from, and write to network connections without needing to manually construct packets. A single TCP connection is uniquely identified by a **socket pair**: (source IP, source port, destination IP, destination port, protocol) — which is exactly what a stateful firewall or a connection-tracking table (like Linux's `conntrack`) uses to distinguish one flow from another, even when many connections share the same destination IP and port.

### Connection-oriented vs. connectionless communication

TCP is **connection-oriented**: before any application data is exchanged, both sides perform a handshake to establish shared state (sequence numbers, window sizes), and the connection is explicitly torn down afterward. UDP is **connectionless**: a datagram is simply sent to a destination address and port with no prior negotiation, no guarantee of delivery, and no ordering guarantee. Choosing between them is a real engineering trade-off covered in depth in the [Networking Core Protocols](../networking-core-protocols/README.md) guide — connection-oriented protocols trade overhead and latency for reliability; connectionless protocols trade reliability for speed and simplicity, which is why DNS, streaming, and VoIP frequently use UDP.

## Why It Matters for Security

- **Firewall rules are fundamentally socket-based** — most traditional firewalls filter on (source IP, destination IP, destination port, protocol), which is exactly the socket concept described above; understanding it is a prerequisite for reading or writing firewall policy.
- **Port scanning** (e.g., with `nmap`) is a direct application of the well-known port registry — an open TCP port 3389 is a strong signal of an exposed RDP service, regardless of what a banner claims.
- **Layer confusion in defensive tooling** is a real gap: a network-layer firewall has no visibility into application-layer (Layer 7) attacks like SQL injection or XSS, which is why web application firewalls (WAFs) exist as a distinct, higher-layer control.
- **Source port randomisation** is itself a security control — predictable ephemeral source ports historically made off-path TCP injection and DNS cache poisoning attacks easier, which is why modern operating systems and resolvers randomise them (see RFC 6056 for TCP port randomisation guidance).

## Common Pitfalls / Misconfigurations

- **Treating OSI as literal reality** — expecting real protocols to cleanly fit into exactly one OSI layer, when in practice several protocols (e.g., TLS) span or blur layer boundaries.
- **Confusing "port closed" with "port filtered"** — a firewall dropping traffic silently versus a host actively refusing a connection produce different scan results and imply different security postures.
- **Exposing management ports beyond intended scope** — binding an admin interface to `0.0.0.0` (all interfaces) instead of a specific internal IP, making it reachable from networks it was never meant to be exposed to.
- **Assuming a non-standard port is obscurity-as-security** — moving SSH from port 22 to port 2222 reduces automated noise but provides no protection against a targeted scan of the full port range.

## Related TryHackMe Rooms in This Series

- [OSI Model](../../fundamentals/osi-model/README.md) — a deeper, dedicated treatment of the seven-layer model summarised here.
- [What is Networking?](../../fundamentals/what-is-networking/README.md) — introduces addressing (IP, MAC, port) that this room formalises into sockets.
- [Networking Essentials](../networking-essentials/README.md) — applies these concepts to DHCP, subnetting, and gateway routing.
- [Networking Core Protocols](../networking-core-protocols/README.md) — examines TCP and UDP headers in detail, building directly on the transport-layer concepts here.

## References

- ISO/IEC 7498-1, Information technology — Open Systems Interconnection — Basic Reference Model — https://www.iso.org/standard/20269.html
- RFC 1122, Requirements for Internet Hosts — Communication Layers — https://www.rfc-editor.org/rfc/rfc1122
- RFC 9293, Transmission Control Protocol (TCP) — https://www.rfc-editor.org/rfc/rfc9293
- RFC 791, Internet Protocol — https://www.rfc-editor.org/rfc/rfc791
- RFC 6056, Recommendations for Transport-Protocol Port Randomization — https://www.rfc-editor.org/rfc/rfc6056
- IANA, Service Name and Transport Protocol Port Number Registry — https://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xhtml
- MDN Web Docs, "How does the Internet work?" — https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Networking/How_does_the_Internet_work
