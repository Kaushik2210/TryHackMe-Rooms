# What is Networking?

**Platform:** TryHackMe · **Type:** Concept Guide (no personal run captured — see note below)
**Primary domain:** Networking

> **Note:** This is a concept guide covering the material this TryHackMe room teaches, written from
> public documentation and reference material. It is not a personal walkthrough — no session evidence
> (screenshots, command output, room-specific answers) is included.

## Overview

Computer networking is the practice of connecting devices so they can exchange data, share resources, and coordinate work. Every technology a security professional touches — from a corporate LAN to the public internet to a cloud VPC — is built on the same small set of ideas: addressing, routing, and protocols that define how bits get from one machine to another. TryHackMe's "What is Networking?" room is the entry point of the Networking learning path, introducing the vocabulary (host, node, link, protocol) and the basic shape of a network before later rooms dig into the OSI model, LANs, and specific protocols. Understanding these fundamentals matters for security work because nearly every attack technique — sniffing, spoofing, lateral movement, exfiltration — is really an abuse of a normal networking mechanism.

## Core Concepts

### What a network actually is

A computer network is a collection of **nodes** (computers, phones, printers, routers, IoT devices — anything with a network interface) connected by **links** (physical media like copper or fibre, or wireless spectrum) that allow them to exchange data using an agreed-upon set of rules called a **protocol**. A protocol specifies the format of messages, the order in which they are sent, and how each side should react — analogous to a shared language and etiquette that lets two strangers hold a conversation.

### Types of networks by scale

Networks are commonly classified by their physical/geographic scope:

| Type | Typical scope | Example |
|---|---|---|
| PAN (Personal Area Network) | A few metres | Bluetooth earbuds paired to a phone |
| LAN (Local Area Network) | A building or campus | Office Wi-Fi and switched Ethernet |
| MAN (Metropolitan Area Network) | A city | A university connecting several campuses |
| WAN (Wide Area Network) | Countries/continents | The internet itself, or an MPLS backbone linking branch offices |

The LAN is the unit most security engagement starts with — internal penetration tests, Active Directory attacks, and most of TryHackMe's practical rooms operate inside a simulated LAN.

### Clients, servers, and peers

Most network communication follows a **client–server model**: a server process listens on a known address/port and responds to requests from clients (e.g., a web browser requesting a page from a web server on TCP/80 or TCP/443). Some systems instead use a **peer-to-peer (P2P) model**, where nodes act as both client and server to each other (e.g., BitTorrent). Recognising which model an application uses matters when mapping an attack surface — a server is a fixed, discoverable target; a peer can appear and disappear.

### Addressing: how a node is found

For two nodes to exchange data, each needs an address that uniquely identifies it on the network:

- **MAC address** — a 48-bit identifier burned into (or spoofable on) a network interface card, used for delivery on the local link (Layer 2).
- **IP address** — a logical address (IPv4, 32 bits, e.g. `192.168.1.10`, or IPv6, 128 bits) used for delivery across networks (Layer 3). IPv4 address space is defined in RFC 791; private, non-routable ranges are defined in RFC 1918 (`10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`).
- **Port number** — a 16-bit value (0–65535) identifying a specific application or service on a host, standardised by IANA. Well-known ports (0–1023) are reserved for common services such as HTTP (80), HTTPS (443), SSH (22), and DNS (53).

These three address types map roughly onto the layered model covered in the [OSI Model](../osi-model/README.md) guide: MAC at Layer 2, IP at Layer 3, ports at Layer 4.

### Protocols: the rules of the conversation

A protocol suite is a stack of protocols, each responsible for a narrower job, that together move an application's data across a network. The internet runs primarily on the **TCP/IP suite** (also called the Internet Protocol Suite, documented across many RFCs, foundationally RFC 791 for IP and RFC 793 for TCP). Key properties every protocol must define:

- **Message format** — header fields, field sizes, encoding.
- **Semantics** — what each message means and what response is expected.
- **Timing/sequencing** — how retransmission, ordering, and flow control are handled.
- **Error handling** — what happens when a message is malformed, lost, or arrives out of order.

### Circuit switching vs. packet switching

Early telephone networks used **circuit switching**: a dedicated path was reserved for the full duration of a call. The internet instead uses **packet switching**: data is broken into discrete packets, each independently routed toward the destination and potentially taking different paths, then reassembled at the far end. Packet switching makes far more efficient use of shared links and is inherently resilient to a single link failure, at the cost of variable latency (jitter) and the need for reassembly/reordering logic at the endpoints. This distinction underlies the whole design of IP-based networks and is expanded on in [Packets & Frames](../packets-and-frames/README.md).

### Bandwidth, throughput, and latency

Three terms are frequently conflated:

- **Bandwidth** — the theoretical maximum data rate of a link (e.g., a 1 Gbps Ethernet port).
- **Throughput** — the actual achieved data rate, which is always ≤ bandwidth due to overhead, congestion, and errors.
- **Latency** — the time for a single bit/packet to travel from source to destination, often measured as round-trip time (RTT) with tools like `ping` (which uses ICMP, RFC 792).

## Why It Matters for Security

Networking fundamentals are the foundation every offensive and defensive technique sits on:

- **Reconnaissance** relies on understanding addressing (identifying live hosts and open ports) — this is exactly what tools like `nmap` automate.
- **Sniffing and eavesdropping** exploit the fact that, on shared media or misconfigured switches, traffic not addressed to you can still be observed.
- **Spoofing attacks** (MAC spoofing, IP spoofing, ARP spoofing) work because many core protocols were designed for a trusted, cooperative network and perform little to no sender authentication.
- **Segmentation and firewalling**, a primary defensive control, is only meaningful once you understand how addressing and routing decide which traffic can reach which node.
- Incident responders need to reconstruct "who talked to whom" from packet captures and flow logs, which requires fluency in the addressing and protocol concepts introduced here.

## Common Pitfalls / Misconfigurations

- **Flat networks** — treating an entire organisation as one LAN with no segmentation means a single compromised host can reach everything else at Layer 2/3.
- **Confusing private and public addressing** — exposing RFC 1918 address space directly to the internet, or relying on NAT alone as a "security boundary" rather than a firewall.
- **Ignoring port scope** — leaving management interfaces (RDP/3389, WinRM/5985-5986, database ports) reachable from untrusted networks because "it's just an internal IP."
- **Assuming physical/local access is safe** — many foundational protocols (ARP, DHCP, plain HTTP) trust anything on the local link, so "it's only on the LAN" is not a security boundary.

## Related TryHackMe Rooms in This Series

- [Intro to LAN](../intro-to-lan/README.md) — goes deeper into how devices communicate within a local network.
- [OSI Model](../osi-model/README.md) — the formal layered framework referenced throughout this guide.
- [Packets & Frames](../packets-and-frames/README.md) — the actual units of data described here as "packets."
- [Extending Your Network](../extending-your-network/README.md) — how LANs connect to larger networks and the internet.
- [Networking Concepts](../../easy/networking-concepts/README.md) and [Networking Essentials](../../easy/networking-essentials/README.md) — build on these basics with more applied detail.

## References

- IANA, Service Name and Transport Protocol Port Number Registry — https://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xhtml
- RFC 791, Internet Protocol — https://www.rfc-editor.org/rfc/rfc791
- RFC 793, Transmission Control Protocol — https://www.rfc-editor.org/rfc/rfc793
- RFC 792, Internet Control Message Protocol — https://www.rfc-editor.org/rfc/rfc792
- RFC 1918, Address Allocation for Private Internets — https://www.rfc-editor.org/rfc/rfc1918
- Cisco, "What Is a Computer Network?" — https://www.cisco.com/c/en/us/solutions/automation/what-is-a-computer-network.html
- MDN Web Docs, "How does the Internet work?" — https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Networking/How_does_the_Internet_work
