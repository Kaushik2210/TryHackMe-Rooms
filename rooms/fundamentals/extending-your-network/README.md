# Extending Your Network

**Platform:** TryHackMe · **Type:** Concept Guide (no personal run captured — see note below)
**Primary domain:** Networking

> **Note:** This is a concept guide covering the material this TryHackMe room teaches, written from
> public documentation and reference material. It is not a personal walkthrough — no session evidence
> (screenshots, command output, room-specific answers) is included.

## Overview

A single Ethernet cable connects two devices; everything past that requires infrastructure that repeats, filters, or routes traffic between many devices and many networks. TryHackMe's "Extending Your Network" room covers the hardware that makes a real LAN — and its connection to the wider internet — possible: repeaters, hubs, switches, routers, and access points, along with the logical techniques (VLANs, NAT) used to segment and conserve address space. This hardware is where most of the practical attack surface security professionals care about actually lives, because these are the devices making forwarding decisions about every frame and packet on the network.

## Core Concepts

### Repeaters and hubs: Layer 1

A **repeater** simply regenerates and amplifies a signal so it can travel further without degrading — a pure physical-layer (Layer 1) device with no understanding of frames or addresses. A **hub** is effectively a multi-port repeater: any electrical signal received on one port is repeated out to every other port. Because a hub has no concept of a MAC address table, every device connected to it shares a single **collision domain** — only one device can transmit at a time without frames colliding, and every device sees every other device's traffic. Hubs are effectively obsolete in modern networks but understanding them matters because their broadcast-everything behaviour is exactly what tools like Wireshark can exploit when they *are* still encountered (or emulated by a network tap).

### Switches: Layer 2

A **switch** operates at the link layer (Layer 2) and makes intelligent forwarding decisions based on MAC addresses. It builds and maintains a **MAC address table** (also called a CAM — Content Addressable Memory — table) by observing the source MAC address of every frame it receives on each port. When a frame arrives for a destination MAC already in the table, the switch forwards it only out the correct port, rather than flooding it everywhere — this means each switch port is its own collision domain, dramatically improving both performance and confidentiality on the local segment compared to a hub. If the destination MAC is unknown, or the frame is a broadcast (destination `FF:FF:FF:FF:FF:FF`), the switch floods it out every port except the one it arrived on. All devices connected to (a set of interconnected) switches still share a single **broadcast domain** unless VLANs are used to divide it.

### Routers: Layer 3

A **router** operates at the network layer (Layer 3) and forwards **packets** between different networks based on destination IP address, using a **routing table** that maps destination network prefixes to next-hop addresses and outgoing interfaces. Where a switch's job is "get this frame to the right MAC address on my LAN," a router's job is "get this packet to the right network, possibly several networks away." Routers are the devices that make the internet an internet — a mesh of independently administered networks connected by routers exchanging reachability information via routing protocols such as OSPF or BGP (RFC 4271 for BGP-4). Every router decrements the IP TTL field and, if it drops below zero, replies with an ICMP Time Exceeded message — the mechanism `traceroute` relies on to enumerate the path to a destination.

### Wireless access points

A **wireless access point (AP)** bridges wireless clients (using Wi-Fi, standardised as IEEE 802.11) onto a wired network, functioning conceptually like a switch port for radio-connected devices. Consumer "routers" are usually a single physical box combining a router, a switch, and an access point, which is a common source of confusion — architecturally, these are three distinct logical functions.

### VLANs: segmenting a broadcast domain

A **Virtual LAN (VLAN)**, standardised by IEEE 802.1Q, lets a single physical switch (or set of switches) be logically partitioned into multiple isolated broadcast domains, as if they were separate physical switches. Each VLAN is identified by a 12-bit VLAN ID (allowing up to 4094 usable VLANs) tagged into the Ethernet frame via an inserted 4-byte 802.1Q tag between the source MAC address and EtherType fields. A port carrying traffic for a single VLAN is an **access port**; a port carrying tagged traffic for multiple VLANs between switches (or to a router) is a **trunk port**. VLANs are the standard tool for segmenting a network by function or trust level — separating, for example, corporate workstations, guest Wi-Fi, and IoT devices onto isolated broadcast domains that can only communicate through a router or firewall enforcing policy between them.

### NAT: stretching a scarce address space

**Network Address Translation (NAT)**, described generally in RFC 3022, lets many devices on a private network (using RFC 1918 address space) share a single public IPv4 address when communicating with the internet. The most common form, **NAPT (Network Address and Port Translation)** — often just called "NAT" or PAT — rewrites both the source IP address and source port of outgoing packets, maintaining a translation table so return traffic can be mapped back to the correct internal host. NAT was originally a stopgap for IPv4 address exhaustion, but it has an incidental security effect: hosts behind NAT are not directly addressable from the internet, which is sometimes (incorrectly) treated as equivalent to a firewall.

## Why It Matters for Security

- **VLAN hopping** attacks (double-tagging attacks, or abusing a switch port left in dynamic trunking mode) let an attacker on one VLAN send traffic into another VLAN it should be isolated from, defeating the segmentation VLANs are meant to provide.
- **MAC flooding** attacks exhaust a switch's MAC address table, forcing it into "fail-open" behaviour where it floods all frames to all ports like a hub — turning a switched network back into one large collision/sniffing domain.
- **Rogue access points and evil twins** exploit the fact that Wi-Fi association is, by default, based only on an SSID name a client trusts, letting an attacker impersonate a legitimate AP to intercept traffic.
- **Misconfigured trunk ports** left enabled on user-facing switch ports (instead of restricted to inter-switch links) are a common lateral-movement vector, since an attacker with physical or logical access can potentially tag traffic into VLANs they shouldn't reach.
- **NAT is not a firewall** — relying on NAT alone for perimeter security ignores that outbound connections, application-layer attacks, and any explicitly forwarded port bypass NAT's incidental protection entirely.

## Common Pitfalls / Misconfigurations

- **Leaving unused switch ports in the default VLAN**, active and untagged, gives any device plugged into an empty wall jack immediate access to the internal network.
- **Native VLAN misconfiguration on trunks** — using the default (often VLAN 1) native VLAN on a trunk port is what makes double-tagging VLAN hopping attacks possible.
- **Flat guest Wi-Fi** — placing a guest wireless network on the same VLAN/broadcast domain as internal corporate devices, defeating the purpose of having a "guest" network at all.
- **Over-reliance on port security defaults** — not configuring port security (limiting or pinning allowed MAC addresses per switch port) leaves switches open to MAC flooding and spoofing.
- **Assuming router = firewall** — a router with only default, permissive routing and no explicit access control lists or stateful filtering enforces no security policy between the networks it connects.

## Related TryHackMe Rooms in This Series

- [What is Networking?](../what-is-networking/README.md) and [Intro to LAN](../intro-to-lan/README.md) — cover the addressing and LAN fundamentals this room's hardware operates on.
- [Packets & Frames](../packets-and-frames/README.md) — the data units these devices are actually forwarding.
- [OSI Model](../osi-model/README.md) — clarifies why repeaters, switches, and routers are described as Layer 1, 2, and 3 devices respectively.
- [Networking Essentials](../../easy/networking-essentials/README.md) — builds on routing and gateway concepts introduced here.

## References

- IEEE 802.1Q, Bridges and Bridged Networks (VLAN tagging standard) — https://standards.ieee.org/ieee/802.1Q/6844/
- IEEE 802.11 Wireless LAN standard overview — https://www.ieee802.org/11/
- RFC 3022, Traditional IP Network Address Translator (Traditional NAT) — https://www.rfc-editor.org/rfc/rfc3022
- RFC 1918, Address Allocation for Private Internets — https://www.rfc-editor.org/rfc/rfc1918
- RFC 4271, A Border Gateway Protocol 4 (BGP-4) — https://www.rfc-editor.org/rfc/rfc4271
- Cisco, "What Is a Switch vs a Router?" — https://www.cisco.com/c/en/us/products/switches/what-is-a-network-switch.html
- Cisco, "What is a VLAN?" — https://www.cisco.com/c/en/us/solutions/enterprise-networks/what-is-a-vlan.html
