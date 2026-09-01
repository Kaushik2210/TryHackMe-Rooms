# OSI Model

**Platform:** TryHackMe · **Type:** Concept Guide (no personal run captured — see note below)
**Primary domain:** Networking

> **Note:** This is a concept guide covering the material this TryHackMe room teaches, written from
> public documentation and reference material. It is not a personal walkthrough — no session evidence
> (screenshots, command output, room-specific answers) is included.

## Overview

The Open Systems Interconnection (OSI) model is a seven-layer conceptual framework, standardised by ISO/IEC 7498-1, that describes how network communication is broken into distinct responsibilities — from raw signalling on a wire up to the data an application actually cares about. TryHackMe's "OSI Model" room teaches this framework because it is the shared vocabulary the entire security and networking industry uses to describe *where* a protocol, device, or attack operates. Saying "ARP spoofing is a Layer 2 attack" or "this is a Layer 7 firewall" only makes sense once you know what each layer is responsible for.

## Core Concepts

### The seven layers

| # | Layer | Responsibility | Example protocols/units |
|---|---|---|---|
| 7 | Application | Provides network services directly to end-user applications | HTTP, DNS, SMTP, FTP — data |
| 6 | Presentation | Translates/encodes data (encryption, compression, character sets) | TLS encoding, JPEG, ASCII/UTF-8 — data |
| 5 | Session | Establishes, manages, and tears down sessions between applications | NetBIOS, RPC, sockets/session tokens — data |
| 4 | Transport | End-to-end delivery, segmentation, reliability, flow control | TCP, UDP — segments/datagrams |
| 3 | Network | Logical addressing and routing between networks | IP, ICMP, routers — packets |
| 2 | Data Link | Node-to-node delivery on the same physical segment, addressing via MAC | Ethernet, Wi-Fi (802.11), switches, ARP — frames |
| 1 | Physical | Raw bit transmission over a medium | Cabling, radio, voltage/light signalling — bits |

A common mnemonic for remembering the order top-down is "**A**ll **P**eople **S**eem **T**o **N**eed **D**ata **P**rocessing" (Application → Physical).

### How data moves through the stack (encapsulation)

When an application sends data, each layer going down the stack wraps ("encapsulates") the data from the layer above with its own header (and sometimes trailer):

```
Application data
  -> [Transport header | data]                       = Segment (TCP) / Datagram (UDP)
    -> [Network header | Transport segment]           = Packet
      -> [Data Link header | Network packet | trailer]= Frame
        -> raw bits on the medium
```

At the receiving end, the process reverses ("decapsulation"): each layer strips its own header and passes the remainder up to the layer above. This layered independence is the core engineering idea of OSI — a Layer 3 router doesn't need to understand HTTP, and a web server doesn't need to know whether it's reachable over Ethernet or Wi-Fi.

### OSI vs. TCP/IP model

In practice, the internet runs on the **TCP/IP (Internet Protocol Suite)** model, which predates and is simpler than OSI — commonly described in four layers (Link, Internet, Transport, Application) that map loosely onto OSI's seven. OSI is taught primarily as a *teaching and diagnostic* framework, not because real stacks implement all seven layers distinctly; Session and Presentation layer functions, for example, are often absorbed into the application itself (e.g., TLS in HTTPS handles what OSI calls Presentation-layer encryption, and application logic manages sessions).

### Per-layer detail relevant to security

- **Layer 1 (Physical):** concerned with cabling, connectors, signalling. Attacks here are physical — wiretapping, cable tapping, RF jamming, rogue hardware implants.
- **Layer 2 (Data Link):** MAC addressing, switching, VLANs, ARP/NDP. Attacks: ARP spoofing, MAC flooding, VLAN hopping, 802.1X bypass. Covered in depth in [Intro to LAN](../intro-to-lan/README.md).
- **Layer 3 (Network):** IP addressing and routing, ICMP. Attacks: IP spoofing, routing manipulation (BGP hijacking is a real-world Layer 3 abuse), ICMP-based reconnaissance and covert channels.
- **Layer 4 (Transport):** TCP/UDP, ports, the three-way handshake, flow/congestion control. Attacks: SYN floods, port scanning, TCP session hijacking, UDP amplification (used in many DDoS attacks, e.g., DNS/NTP/memcached reflection).
- **Layer 5 (Session):** session establishment/teardown. Attacks: session hijacking, session fixation (often discussed at this conceptual layer even though implementation lives in the application).
- **Layer 6 (Presentation):** encoding/encryption/compression. Attacks: downgrade attacks against TLS, encoding-based evasion (e.g., double URL-encoding to bypass a filter).
- **Layer 7 (Application):** the actual protocol logic end users rely on. Attacks: SQL injection, XSS, HTTP request smuggling, DNS cache poisoning — the layer where the overwhelming majority of modern application-level attacks occur.

### Devices mapped to layers

- **Hub/repeater** — Layer 1 (regenerates signal, no addressing awareness).
- **Switch/bridge** — Layer 2 (forwards by MAC address); "Layer 3 switches" also perform routing.
- **Router** — Layer 3 (forwards by IP address between networks).
- **Traditional firewall** — Layer 3/4 (filters by IP/port).
- **Next-Generation Firewall (NGFW) / WAF** — Layer 7 (inspects application content, e.g., HTTP requests).

## Why It Matters for Security

The OSI model gives defenders and attackers a common way to scope a problem. Network segmentation controls (VLANs, firewalls) are described by the layer they operate at, which directly determines what they can and cannot stop — a Layer 3/4 firewall blocking a port number cannot detect a SQL injection payload riding legitimately over an allowed port, because that requires Layer 7 inspection. Similarly, packet capture and analysis tools (Wireshark, tcpdump) present traffic layer-by-layer, and being able to say "this anomaly is happening at Layer 2, not Layer 3" is often the fastest way to scope an incident (e.g., distinguishing an ARP spoofing attack from a routing misconfiguration). Vulnerability classes also map cleanly to layers, which is why frameworks like the OWASP Top 10 focus almost entirely on Layer 7 while network penetration testing methodologies (e.g., PTES) explicitly separate Layer 2/3 attacks from application-layer attacks.

## Common Pitfalls / Misconfigurations

- **Layer confusion in defenses** — deploying only a Layer 3/4 firewall and assuming it protects against Layer 7 threats like injection attacks or malicious file uploads.
- **Treating OSI as if it were the literal implementation** — real stacks (like TCP/IP) don't cleanly implement all seven layers; over-insisting on OSI purity can obscure how, e.g., TLS actually spans what OSI calls Presentation and Session layer functions inside a Layer 7-adjacent library.
- **Ignoring lower layers during an application-layer investigation** — troubleshooting an HTTPS failure purely at Layer 7 without checking that DNS resolution (Layer 7) and basic IP reachability/MTU issues (Layer 3) aren't the actual root cause.
- **Forgetting that lower-layer trust is often implicit** — many higher-layer protocols implicitly trust that Layer 2/3 delivered the packet to the right place unmodified, which is exactly the assumption ARP/IP spoofing violates.

## Related TryHycMe Rooms in This Series

- [What is Networking?](../what-is-networking/README.md) — introduces the addressing concepts (MAC/IP/port) formalised here by layer.
- [Intro to LAN](../intro-to-lan/README.md) — deep dive on Layer 2 (Ethernet, ARP, switching).
- [Packets & Frames](../packets-and-frames/README.md) — the actual encapsulated units (frames, packets, segments) discussed in this room.
- [Extending Your Network](../extending-your-network/README.md) — Layer 3 routing between networks.
- [Networking Core Protocols](../../easy/networking-core-protocols/README.md) and [Networking Secure Protocols](../../easy/networking-secure-protocols/README.md) — apply this layered model to specific real protocols.

## References

- ISO/IEC 7498-1:1994, Information technology — Open Systems Interconnection — Basic Reference Model — https://www.iso.org/standard/20269.html
- ITU-T Recommendation X.200, OSI Reference Model — https://www.itu.int/rec/T-REC-X.200
- Cloudflare Learning Center, "What is the OSI Model?" — https://www.cloudflare.com/learning/ddos/glossary/open-systems-interconnection-model-osi/
- MDN Web Docs, "How does the Internet work?" — https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Networking/How_does_the_Internet_work
- OWASP Top 10 (application/Layer 7 risk focus) — https://owasp.org/www-project-top-ten/
- RFC 1122, Requirements for Internet Hosts — Communication Layers — https://www.rfc-editor.org/rfc/rfc1122
