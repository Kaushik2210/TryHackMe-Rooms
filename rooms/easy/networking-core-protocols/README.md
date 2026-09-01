# Networking Core Protocols

**Platform:** TryHackMe · **Type:** Concept Guide (no personal run captured — see note below)
**Primary domain:** Networking

> **Note:** This is a concept guide covering the material this TryHackMe room teaches, written from
> public documentation and reference material. It is not a personal walkthrough — no session evidence
> (screenshots, command output, room-specific answers) is included.

## Overview

TCP, UDP, and ICMP are the three transport/network-layer protocols that essentially every other protocol on the internet is built on top of. TryHackMe's "Networking Core Protocols" room examines them at the header-field level: how TCP establishes and maintains a reliable, ordered connection through its three-way handshake and sequence/acknowledgement numbers; how UDP forgoes all of that for speed and simplicity; and how ICMP carries the diagnostic and error-reporting messages (like the ones behind `ping` and `traceroute`) that keep the rest of the network honest. Fluency with these headers is a direct prerequisite for reading packet captures, writing firewall rules, and understanding an enormous share of both attack and defence techniques.

## Core Concepts

### TCP: reliable, connection-oriented delivery

The **Transmission Control Protocol (TCP)** was originally specified in RFC 793 and is now consolidated in RFC 9293, which formally obsoletes the original. A TCP header (minimum 20 bytes) includes:

- **Source port / Destination port** (16 bits each) — identify the sending and receiving application.
- **Sequence number** (32 bits) — the byte offset of this segment's data within the overall stream, used to reorder segments that arrive out of order.
- **Acknowledgement number** (32 bits) — the next sequence number the sender expects to receive, confirming receipt of everything before it.
- **Flags** — control bits including SYN (synchronise, start a connection), ACK (acknowledge), FIN (finish, gracefully close), RST (reset, abort abnormally), PSH (push data to the application immediately), and URG (urgent pointer valid).
- **Window size** (16 bits) — how many bytes the sender is currently willing to receive, used for flow control.
- **Checksum** — covers the TCP header, data, and a pseudo-header derived from the IP addresses, protecting against corruption.

### The TCP three-way handshake

Before any application data flows, TCP performs a three-step handshake to synchronise sequence numbers and confirm both sides are ready:

1. **SYN** — the client sends a segment with the SYN flag set and an initial sequence number (ISN), e.g., `SEQ=x`.
2. **SYN-ACK** — the server responds with SYN and ACK set, its own ISN (`SEQ=y`), and acknowledges the client's ISN (`ACK=x+1`).
3. **ACK** — the client responds with ACK set and `ACK=y+1`, completing the handshake; the connection is now established (`ESTABLISHED` state) and data can flow.

Closing a TCP connection is a separate four-step process (FIN, ACK, FIN, ACK) since either side can independently signal it has no more data to send while still being willing to receive — this is why a "half-closed" connection state exists. An abrupt **RST** flag, by contrast, tears the connection down immediately without this graceful exchange, and is what a host sends when it receives a segment for a connection it has no record of (e.g., a SYN-ACK to a port nothing is listening on triggers an immediate RST) — the exact mechanic TCP port scanners rely on to distinguish open, closed, and filtered ports.

### UDP: minimal, connectionless delivery

The **User Datagram Protocol (UDP)**, defined in RFC 768, is dramatically simpler. Its header is a fixed 8 bytes:

- **Source port** / **Destination port** (16 bits each)
- **Length** (16 bits) — the length of the UDP header plus data
- **Checksum** (16 bits) — optional in IPv4 (mandatory in IPv6), covering the header, data, and a pseudo-header

UDP provides no handshake, no acknowledgement, no retransmission, and no ordering guarantee — a datagram is simply sent, and it's entirely up to the application layer to handle loss or reordering if it needs to. This trade-off is deliberate: protocols where low latency matters more than perfect delivery (DNS queries, VoIP, live video, online gaming, and DHCP) use UDP because retransmission delay would be more harmful than an occasional dropped or duplicated packet.

### ICMP: diagnostics and error reporting

The **Internet Control Message Protocol (ICMP)**, defined in RFC 792, is not a transport protocol for application data at all — it's the mechanism IP itself uses to report errors and provide network diagnostics. Every ICMP message has a **Type** and **Code** field identifying what happened. The two most operationally significant are:

- **Echo Request (Type 8) / Echo Reply (Type 0)** — the basis of `ping`, used to test basic reachability and measure round-trip latency.
- **Time Exceeded (Type 11)** — sent by a router when a packet's TTL reaches zero; `traceroute` deliberately sends packets with incrementing TTLs (starting at 1) specifically to elicit one of these messages from each router along the path, mapping the route hop by hop.
- **Destination Unreachable (Type 3)**, with codes for host unreachable, port unreachable, and (notably) fragmentation-needed-but-DF-set — the message Path MTU Discovery relies on to learn the smallest MTU along a path.

Because ICMP carries no application data and has historically been treated as "just diagnostics," it is frequently either blocked wholesale by overcautious firewalls (breaking Path MTU Discovery in the process) or left completely unfiltered (enabling reconnaissance and covert channels).

### Choosing TCP vs. UDP

| | TCP | UDP |
|---|---|---|
| Connection setup | Three-way handshake | None |
| Reliability | Guaranteed delivery, retransmission | Best-effort, no retransmission |
| Ordering | Guaranteed | Not guaranteed |
| Overhead | Higher (headers, ACKs, state) | Minimal |
| Typical uses | HTTP/HTTPS, SSH, SMTP, file transfer | DNS, DHCP, VoIP, streaming, online games |

## Why It Matters for Security

- **Port scanning techniques are direct exploitations of these header behaviours** — a TCP SYN scan (`nmap -sS`) sends only a SYN and reads the response (SYN-ACK = open, RST = closed, no response/ICMP unreachable = filtered) without ever completing the handshake, making it faster and historically stealthier than a full connect scan.
- **SYN flood attacks** abuse the handshake's asymmetry: a server allocates state after receiving a SYN but before the handshake completes, so flooding a target with SYNs (often from spoofed source IPs) can exhaust its connection table — the classic defence is SYN cookies, which defer state allocation until the final ACK.
- **UDP's lack of a handshake makes source IP spoofing trivial** for UDP-based traffic, which is exactly what enables UDP-based amplification/reflection DDoS attacks (e.g., abusing open DNS or NTP servers to send large responses to a spoofed victim address).
- **ICMP tunneling and covert channels** exploit ICMP being commonly permitted through firewalls even when other protocols are blocked, encoding data in Echo Request/Reply payloads to exfiltrate data or establish command-and-control.
- **Firewall rule design depends on flag-level TCP awareness** — a stateful firewall permits inbound SYN-ACK/ACK only in response to an outbound SYN it tracked, while a stateless (or badly configured) firewall might permit any ACK-flagged packet inbound, an old but instructive ACK-scan evasion technique.

## Common Pitfalls / Misconfigurations

- **Blocking all ICMP** at the perimeter, which breaks Path MTU Discovery and legitimate diagnostics, often causing confusing "connection hangs" for large payloads rather than a clean error.
- **Allowing UDP responses without egress filtering**, letting internal misconfigured services participate unwittingly in reflection/amplification attacks against third parties.
- **Relying on source port/IP alone for trust decisions**, given how easily both are spoofed in UDP traffic and, to a lesser extent, forged in crafted TCP packets before a handshake completes.
- **Missing SYN flood protections** on internet-facing TCP services — not enabling SYN cookies or connection rate limiting leaves a service needlessly exposed to a well-understood, decades-old attack class.
- **Confusing "filtered" with "closed" in scan results** — treating a lack of ICMP unreachable response as proof a port is closed, when it may simply mean a firewall silently dropped the probe.

## Related TryHackMe Rooms in This Series

- [Packets & Frames](../../fundamentals/packets-and-frames/README.md) — the IP packet structure that TCP, UDP, and ICMP are all encapsulated inside.
- [Networking Concepts](../networking-concepts/README.md) — the port and socket vocabulary this room's header fields build directly on.
- [Networking Essentials](../networking-essentials/README.md) — DHCP (UDP-based) and gateway routing (which relies on ICMP for diagnostics) covered in practical context.
- [Networking Secure Protocols](../networking-secure-protocols/README.md) — how TLS is layered on top of TCP to add confidentiality and integrity to these same connections.

## References

- RFC 9293, Transmission Control Protocol (TCP) — https://www.rfc-editor.org/rfc/rfc9293
- RFC 793, Transmission Control Protocol (original specification, obsoleted by RFC 9293) — https://www.rfc-editor.org/rfc/rfc793
- RFC 768, User Datagram Protocol — https://www.rfc-editor.org/rfc/rfc768
- RFC 792, Internet Control Message Protocol — https://www.rfc-editor.org/rfc/rfc792
- RFC 4987, TCP SYN Flooding Attacks and Common Mitigations — https://www.rfc-editor.org/rfc/rfc4987
- Nmap Reference Guide, "Port Scanning Techniques" — https://nmap.org/book/man-port-scanning-techniques.html
- IANA, Internet Control Message Protocol (ICMP) Parameters — https://www.iana.org/assignments/icmp-parameters/icmp-parameters.xhtml
