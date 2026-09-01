# Intro to LAN

**Platform:** TryHackMe · **Type:** Concept Guide (no personal run captured — see note below)
**Primary domain:** Networking

> **Note:** This is a concept guide covering the material this TryHackMe room teaches, written from
> public documentation and reference material. It is not a personal walkthrough — no session evidence
> (screenshots, command output, room-specific answers) is included.

## Overview

A Local Area Network (LAN) is a network confined to a single site — an office, home, or campus — where devices are typically connected by Ethernet switches and/or Wi-Fi access points under one administrative control. TryHackMe's "Intro to LAN" room builds on general networking concepts to explain how devices inside the same physical or logical network actually find and talk to each other: MAC addressing, Ethernet switching, ARP, and basic topology. This is the layer where the vast majority of internal penetration testing and lateral-movement techniques operate, so understanding it precisely is a prerequisite for almost every offensive networking skill.

## Core Concepts

### LAN building blocks

- **Network Interface Card (NIC)** — hardware that connects a device to the network medium and holds a unique MAC address.
- **Switch** — a Layer 2 device that forwards Ethernet frames only to the port where the destination MAC address lives, learned dynamically by inspecting the source MAC of incoming frames (this is the "MAC address table" or CAM table).
- **Hub** (largely obsolete) — a "dumb" Layer 1 device that repeats every incoming signal out of every other port, meaning all connected hosts share one collision domain and can see all traffic — this is why old-style shared Ethernet was trivially sniffable.
- **Access Point (AP)** — a bridge between wired Ethernet and 802.11 wireless clients, effectively extending the same LAN over radio.

### MAC addressing

Every NIC has a 48-bit **Media Access Control (MAC) address**, typically written as six hex octets (e.g. `00:1A:2B:3C:4D:5E`). The first three octets (the OUI, Organizationally Unique Identifier) identify the manufacturer and are registered with the IEEE. MAC addresses are used for delivery *within* a single Layer 2 broadcast domain — a switch uses the destination MAC to decide which physical port to forward a frame out of. Unlike IP addresses, MAC addresses are not designed to be globally routable; they only need to be unique on the local segment. They can also be changed in software ("MAC spoofing") since the OS controls what value the NIC driver reports.

### Ethernet framing basics

Ethernet (IEEE 802.3) is the dominant wired LAN standard. An Ethernet II frame consists of: destination MAC (6 bytes), source MAC (6 bytes), EtherType (2 bytes, identifying the payload protocol, e.g. `0x0800` for IPv4 or `0x0806` for ARP), a payload (46–1500 bytes for the standard MTU), and a trailing 4-byte Frame Check Sequence (CRC-32) for error detection. This is examined in more depth in [Packets & Frames](../packets-and-frames/README.md).

### Address Resolution Protocol (ARP)

IP-based applications address each other by IP address, but Ethernet delivery needs a destination MAC address. **ARP** (RFC 826) bridges this gap on IPv4 networks:

1. Host A wants to send an IP packet to Host B on the same subnet but doesn't know B's MAC address.
2. A broadcasts an **ARP Request** ("Who has 192.168.1.20? Tell 192.168.1.10") to the broadcast MAC `ff:ff:ff:ff:ff:ff`, so every host on the segment receives it.
3. Host B recognises its own IP and replies with a unicast **ARP Reply** containing its MAC address.
4. Host A caches this mapping in its local **ARP table** (`arp -a` on most OSes) for a limited time to avoid repeating the broadcast for every packet.

ARP has no authentication — any host can claim to own any IP address, and other hosts will believe the reply and cache it. This is the root cause of ARP spoofing/poisoning attacks (see below). IPv6 replaces ARP with **Neighbor Discovery Protocol (NDP)**, defined in RFC 4861, which runs over ICMPv6 and adds some additional mechanisms (like Secure Neighbor Discovery) but is still vulnerable to analogous spoofing in the absence of protections.

### Broadcast domains and collision domains

- A **collision domain** is a set of devices where two simultaneous transmissions can collide (relevant mainly to legacy shared-media Ethernet/hubs; switches create a separate collision domain per port).
- A **broadcast domain** is the set of devices that receive a Layer 2 broadcast frame (destination MAC `ff:ff:ff:ff:ff:ff`). All devices on the same VLAN/subnet share one broadcast domain unless separated by a router or Layer 3 switch. Large broadcast domains create noise and a larger attack/reconnaissance surface (e.g., every host sees every ARP request), which is why segmentation with VLANs is a standard control.

### Topologies

LANs are physically or logically cabled in a handful of common shapes: **bus** (all devices share one cable, now obsolete), **star** (every device connects to a central switch — the dominant modern design), **ring** (each device connects to exactly two neighbours, historically Token Ring/FDDI), and **mesh** (many redundant interconnections, used in resilient backbone and wireless mesh designs). Star topology dominates modern LANs because switch failure or cable damage only isolates one device instead of the whole segment.

### Duplex and speed

Modern switched Ethernet operates in **full duplex** (simultaneous send/receive on separate wire pairs or channels), eliminating collisions entirely on point-to-point switch links. Common LAN speeds are 100 Mbps (Fast Ethernet), 1 Gbps, 2.5/5/10 Gbps (increasingly common on modern NICs), and beyond, standardised under IEEE 802.3.

## Why It Matters for Security

- **ARP spoofing (ARP cache poisoning)** — an attacker sends unsolicited, forged ARP replies claiming to own the gateway's IP address, causing victims to send traffic to the attacker's MAC instead. This enables on-path (man-in-the-middle) interception of LAN traffic and is one of the oldest and most reliable internal attack techniques, implemented by tools like Ettercap and Bettercap.
- **MAC flooding** — an attacker floods a switch with frames using many bogus source MAC addresses to exhaust the CAM table; once full, some switches "fail open" and start behaving like a hub, broadcasting all traffic to every port and enabling passive sniffing.
- **MAC spoofing** — bypassing MAC-based access control lists (common on guest Wi-Fi or NAC systems) by cloning an allowed device's MAC address.
- **Rogue DHCP/AP** — because a LAN broadcast domain trusts anything plugged in or associated, an attacker who gains physical or wireless access can inject a rogue DHCP server or access point to redirect victims' traffic.
- **VLAN hopping** — techniques (double tagging, switch spoofing) that let an attacker on one VLAN reach traffic on another, defeating naive Layer 2 segmentation.

## Common Pitfalls / Misconfigurations

- Relying on **VLANs alone**, with no ACLs/firewalling between them, as if segmentation itself were a security boundary rather than just a broadcast-domain boundary.
- Leaving **switch ports unused and unsecured** (no port security, no 802.1X) so a visitor can simply plug into an empty wall jack and join the internal LAN.
- No **Dynamic ARP Inspection (DAI)** or static ARP entries on high-value segments, leaving ARP spoofing trivially possible.
- Deploying **guest Wi-Fi on the same broadcast domain** as corporate devices instead of an isolated VLAN/SSID.
- Assuming a switched network is immune to sniffing — it dramatically reduces it compared to a hub, but ARP spoofing, port mirroring misuse, and MAC flooding all defeat that assumption.

## Related TryHackMe Rooms in This Series

- [What is Networking?](../what-is-networking/README.md) — the addressing and protocol vocabulary this room builds on.
- [OSI Model](../osi-model/README.md) — places MAC addressing and ARP formally at Layers 2 and 2/3.
- [Packets & Frames](../packets-and-frames/README.md) — the exact structure of the Ethernet frames and ARP messages described here.
- [Extending Your Network](../extending-your-network/README.md) — how multiple LANs are interconnected via routers and WAN links.
- [Networking Core Protocols](../../easy/networking-core-protocols/README.md) — covers DHCP, DNS, and other protocols that ride on top of the LAN.

## References

- RFC 826, An Ethernet Address Resolution Protocol (ARP) — https://www.rfc-editor.org/rfc/rfc826
- RFC 4861, Neighbor Discovery for IP version 6 (IPv6) — https://www.rfc-editor.org/rfc/rfc4861
- IEEE 802.3 Ethernet Standard overview — https://www.ieee802.org/3/
- IEEE Registration Authority, MAC address / OUI lookup — https://standards-oui.ieee.org/
- Cisco, Understanding Switch Port Security — https://www.cisco.com/c/en/us/support/docs/lan-switching/spanning-tree-protocol/10548-62.html
- NIST SP 800-115, Technical Guide to Information Security Testing and Assessment (network sniffing/ARP context) — https://csrc.nist.gov/pubs/sp/800/115/final
