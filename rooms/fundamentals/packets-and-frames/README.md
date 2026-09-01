# Packets & Frames

**Platform:** TryHackMe · **Type:** Concept Guide (no personal run captured — see note below)
**Primary domain:** Networking

> **Note:** This is a concept guide covering the material this TryHackMe room teaches, written from
> public documentation and reference material. It is not a personal walkthrough — no session evidence
> (screenshots, command output, room-specific answers) is included.

## Overview

Every message that crosses a network — a web request, a DNS lookup, a video stream — is broken down and wrapped in layers of headers before it ever touches a wire or a radio signal. TryHackMe's "Packets & Frames" room examines the concrete data units that carry that information: **frames** at the link layer and **packets** at the network layer, plus the segments and datagrams the transport layer wraps them around. Understanding these structures is not academic trivia — it is the difference between reading a packet capture fluently and staring at a wall of hex. Every tool a security engineer relies on, from `tcpdump` to Wireshark to an IDS signature, is ultimately parsing these exact byte layouts.

## Core Concepts

### Encapsulation: data wrapped in data

When an application sends data, each layer of the network stack adds its own header (and sometimes a trailer) around the data handed to it by the layer above — a process called **encapsulation**. The terminology for the resulting unit changes by layer:

| Layer (TCP/IP) | Protocol Data Unit (PDU) | Added by |
|---|---|---|
| Application | Data / message | The application itself |
| Transport | Segment (TCP) / Datagram (UDP) | TCP or UDP header |
| Network/Internet | Packet | IP header |
| Link | Frame | Ethernet (or Wi-Fi) header + trailer |

At the receiving end, the reverse process — **decapsulation** — strips each header off in order as the data climbs back up the stack. This layered wrapping is what lets a single Ethernet frame carry an IP packet, which carries a TCP segment, which carries an HTTP request, without any layer needing to understand the layers above or below it.

### The Ethernet frame

Ethernet, standardised by IEEE 802.3, is the dominant link-layer technology for wired LANs. A standard Ethernet II frame has the following structure:

| Field | Size | Purpose |
|---|---|---|
| Preamble + SFD | 8 bytes | Physical-layer synchronisation (not usually shown by capture tools) |
| Destination MAC address | 6 bytes | Hardware address of the next-hop interface |
| Source MAC address | 6 bytes | Hardware address of the sending interface |
| EtherType | 2 bytes | Identifies the encapsulated protocol (e.g., `0x0800` for IPv4, `0x0806` for ARP, `0x86DD` for IPv6) |
| Payload | 46–1500 bytes | The encapsulated packet (padded if smaller than 46 bytes) |
| Frame Check Sequence (FCS) | 4 bytes | CRC-32 checksum for error detection |

The EtherType field is what lets a network interface hand a frame's payload to the correct next protocol handler — IPv4, IPv6, or ARP, among others — without inspecting the payload itself. Frames are addressed and delivered using **MAC addresses**, 48-bit identifiers assigned to network interface cards, conventionally written as six hex octets (e.g., `00:1A:2B:3C:4D:5E`). The first three octets form the Organisationally Unique Identifier (OUI), assigned by the IEEE to the hardware vendor, which is why MAC addresses can often be used to fingerprint a device manufacturer.

### MTU and fragmentation

The **Maximum Transmission Unit (MTU)** is the largest payload a given link layer can carry in a single frame. Standard Ethernet has an MTU of 1500 bytes; some networks support **jumbo frames** up to around 9000 bytes for higher throughput on data-centre links. When an IP packet is larger than the MTU of a link it needs to cross, it must be split — a process called **fragmentation** — with each fragment carrying its own IP header and a fragment offset so the destination (or, for IPv6, only the originating host) can reassemble the original packet. IPv4 fragmentation is defined in RFC 791; IPv6 removed in-network fragmentation entirely (RFC 8200), pushing the responsibility to the sending host via Path MTU Discovery. Fragmentation adds processing overhead and has historically been abused in attacks (e.g., malformed or overlapping fragments designed to evade intrusion detection or crash naive TCP/IP stacks), which is why many security-conscious networks prefer to avoid it via correctly tuned MTUs.

### The IP packet

Wrapped inside the Ethernet frame's payload is the IP packet. An IPv4 header (RFC 791) is at minimum 20 bytes and includes:

- **Version** (4 bits) — 4 for IPv4, 6 for IPv6.
- **Total Length** — the full packet size including header.
- **Time to Live (TTL)** — decremented by each router hop; when it reaches zero the packet is discarded and an ICMP Time Exceeded message is sent back (RFC 792). TTL is also a common (if imprecise) OS-fingerprinting signal, since Windows, Linux, and various network appliances default to different starting TTL values.
- **Protocol** — identifies the next-layer protocol (6 for TCP, 17 for UDP, 1 for ICMP), per IANA's protocol number registry.
- **Source IP address** and **Destination IP address** — 32-bit addresses identifying the endpoints.
- **Header Checksum** — validates the header only (not the payload), recalculated at every hop because TTL changes.

IPv6 (RFC 8200) simplifies this to a fixed 40-byte base header and moves optional features into extension headers, dropping the header checksum entirely (relying on link-layer and transport-layer checksums instead) and removing router-level fragmentation.

### Frames vs. packets: why the distinction matters

A frame is only meaningful on the local link — a router receiving a frame strips the Ethernet header entirely, inspects the enclosed IP packet, decides the next hop, and re-encapsulates the same packet in a *brand-new* frame addressed to the next hop's MAC address. The IP packet's source and destination addresses stay constant end-to-end (barring NAT), but the frame's source and destination MAC addresses change at every hop. This is a foundational fact for reading packet captures: source/destination MAC tells you about the local segment; source/destination IP tells you about the true endpoints of the conversation.

## Why It Matters for Security

- **Packet capture analysis** (Wireshark, tcpdump, Zeek) is fundamentally the skill of decoding these exact frame and packet structures — every filter (`eth.addr`, `ip.ttl`, `ip.proto`) maps directly to a header field described above.
- **ARP spoofing and MAC flooding** attacks exploit the fact that frame delivery trusts the source MAC address with no authentication, letting an attacker redirect local traffic by lying about which MAC address owns an IP address.
- **Fragmentation-based evasion** (tiny fragments, overlapping fragments) has historically been used to slip malicious traffic past signature-based IDS/IPS systems that fail to reassemble fragments identically to the target host.
- **TTL and header analysis** support both reconnaissance (OS fingerprinting via default TTL and other header quirks) and detection (unusual TTL values can indicate spoofed traffic or a hidden extra hop, such as a covert relay).
- **Covert channels** have been built by encoding data in normally-unused or loosely-specified header fields (IP ID, TTL, TCP options), which is only recognisable to an analyst who knows what "normal" header values look like.

## Common Pitfalls / Misconfigurations

- **MTU mismatches** across a path (common in VPN tunnels, which add their own encapsulation overhead) cause silent packet loss when ICMP "fragmentation needed" messages are blocked by an overly aggressive firewall — a scenario often called a "black hole" MTU problem.
- **Trusting MAC addresses as identity** — MAC addresses are trivially spoofable in software, so access control based solely on MAC filtering (e.g., on Wi-Fi) provides negligible real security.
- **Ignoring jumbo frame compatibility** — enabling jumbo frames on only part of a network path causes large frames to be silently dropped by devices that don't support them, rather than cleanly fragmented.
- **Assuming checksums guarantee integrity** — the IPv4 header checksum and Ethernet FCS catch accidental bit errors, not deliberate tampering; they provide no cryptographic guarantee against an active attacker.

## Related TryHackMe Rooms in This Series

- [What is Networking?](../what-is-networking/README.md) — introduces the addressing concepts (MAC, IP, port) referenced throughout this guide.
- [OSI Model](../osi-model/README.md) — the formal layered framework this room's terminology (frame, packet, segment) maps onto.
- [Extending Your Network](../extending-your-network/README.md) — how the devices that forward these frames and packets (switches, routers) actually work.
- [Networking Core Protocols](../../easy/networking-core-protocols/README.md) — goes deeper into the TCP and UDP headers carried inside these packets.

## References

- RFC 791, Internet Protocol — https://www.rfc-editor.org/rfc/rfc791
- RFC 792, Internet Control Message Protocol — https://www.rfc-editor.org/rfc/rfc792
- RFC 8200, Internet Protocol, Version 6 (IPv6) Specification — https://www.rfc-editor.org/rfc/rfc8200
- IEEE 802.3 Ethernet Standard overview — https://www.ieee802.org/3/
- IANA, Protocol Numbers registry — https://www.iana.org/assignments/protocol-numbers/protocol-numbers.xhtml
- IANA, IEEE 802 Numbers / EtherType registry — https://www.iana.org/assignments/ieee-802-numbers/ieee-802-numbers.xhtml
- MDN Web Docs, "How does the Internet work?" — https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Networking/How_does_the_Internet_work
- Wireshark documentation, "Chapter 4. Terminology" — https://www.wireshark.org/docs/wsug_html_chunked/ChIntroTerminologySection.html
