# Networking Essentials

**Platform:** TryHackMe · **Type:** Concept Guide (no personal run captured — see note below)
**Primary domain:** Networking

> **Note:** This is a concept guide covering the material this TryHackMe room teaches, written from
> public documentation and reference material. It is not a personal walkthrough — no session evidence
> (screenshots, command output, room-specific answers) is included.

## Overview

Getting a device onto a network — and getting its traffic out to the internet — depends on a handful of practical mechanisms that most users never think about: how a device gets an IP address automatically, how that address is understood to belong to a particular subnet, and how traffic destined for anywhere outside that subnet finds its way to a default gateway. TryHackMe's "Networking Essentials" room covers DHCP, subnetting fundamentals, and gateway routing — the everyday machinery that makes "plug in and it just works" possible, and which every network security assessment has to reason about explicitly.

## Core Concepts

### DHCP: automatic address configuration

The **Dynamic Host Configuration Protocol (DHCP)**, defined in RFC 2131, automates the process of assigning an IP address (plus subnet mask, default gateway, and DNS servers) to a device joining a network, removing the need for manual configuration. DHCP operates over UDP, with the server listening on port 67 and the client on port 68. The exchange follows the well-known **DORA** sequence:

1. **Discover** — the client, having no IP address yet, broadcasts a DHCPDISCOVER message to `255.255.255.255` asking if any DHCP server can help.
2. **Offer** — a DHCP server responds with a DHCPOFFER proposing an IP address and lease terms.
3. **Request** — the client broadcasts a DHCPREQUEST formally asking to use the offered address (broadcast so any other DHCP servers that made an offer know it wasn't chosen).
4. **Acknowledge** — the server confirms with a DHCPACK, finalising the lease for a defined duration, after which the client must renew.

Because the initial Discover is a broadcast with no authentication, DHCP is inherently a "first responder wins" protocol on a shared segment — a property that has direct security consequences (see below).

### IP addresses and subnet masks

An IPv4 address is a 32-bit number conventionally written in dotted-decimal notation (e.g., `192.168.1.10`). A **subnet mask** (or, in CIDR notation, a prefix length like `/24`) divides that 32-bit address into a **network portion** and a **host portion**. For example, `192.168.1.0/24` (subnet mask `255.255.255.0`) means the first 24 bits identify the network and the remaining 8 bits identify individual hosts within it, giving 256 total addresses (192.168.1.0 through 192.168.1.255), of which the first (network address) and last (broadcast address) are reserved, leaving 254 usable host addresses. CIDR (Classless Inter-Domain Routing), defined in RFC 4632, replaced the older rigid Class A/B/C system, allowing networks to be sized to actual need rather than forced into fixed 8/16/24-bit boundaries. **Subnetting** — splitting a larger network into smaller subnets by borrowing bits from the host portion for the network portion — is the fundamental tool for both efficient address allocation and traffic segmentation.

### Default gateways and routing

A host on a subnet can talk directly (via Layer 2 / ARP) only to other hosts on the *same* subnet. To reach anything outside that subnet, the host consults its **routing table**: if no more specific route matches the destination, the packet is sent to the **default gateway** — typically a router interface configured as the "exit point" of the local subnet. The gateway then makes its own routing decision, forwarding the packet toward its destination, possibly through several more routers, each performing the same lookup-and-forward step, until it reaches a router directly connected to the destination's network. This hop-by-hop decision-making, rather than any single device knowing the complete path in advance, is the essence of how IP routing scales to the size of the internet.

### How a host decides "local" vs. "remote"

Before sending a packet, a host performs a simple but crucial calculation: it applies its own subnet mask to both its own IP address and the destination IP address. If the resulting network portions match, the destination is assumed to be on the local subnet and the host resolves the destination's MAC address directly via ARP (RFC 826) and sends the frame straight there. If they don't match, the host instead ARPs for the *default gateway's* MAC address and sends the frame there instead, trusting the gateway to route it onward. This single mask comparison is why an incorrect subnet mask configuration can cause a host to either fail to reach local devices (thinking they're remote) or fail to escape the local subnet at all (thinking remote hosts are local).

### DNS as a supporting essential

While DNS is covered in its own dedicated room, it's worth noting here that DHCP typically also hands out DNS server addresses as part of its configuration payload — without this, a host could have full IP connectivity but be unable to resolve any hostname, a common real-world troubleshooting scenario.

## Why It Matters for Security

- **Rogue DHCP servers** exploit DHCP's unauthenticated, first-responder-wins design: an attacker running a rogue DHCP server on a shared segment can hand out a malicious default gateway or DNS server address, silently routing a victim's traffic through an attacker-controlled machine (a classic man-in-the-middle setup).
- **DHCP starvation attacks** flood a legitimate DHCP server with bogus DHCPREQUEST messages using spoofed MAC addresses, exhausting the address pool and denying service to legitimate clients — often as a precursor to standing up a rogue DHCP server once the real one is starved out.
- **Subnetting is the primary tool for network segmentation** — isolating a compromised host in a small subnet with tightly firewalled gateway access limits lateral movement far more effectively than a large flat network.
- **Misrouted or overly broad routing tables** can accidentally expose internal subnets to each other (or to the internet) with no filtering, turning what was intended as a segmentation boundary into a routed-through pathway.
- **DHCP snooping and dynamic ARP inspection**, available on many managed switches, are direct defensive countermeasures to rogue DHCP and ARP-based attacks — they work by having the switch itself track which port is legitimately allowed to answer DHCP or ARP.

## Common Pitfalls / Misconfigurations

- **Oversized subnets** ("just put everything on a /16 to be safe") that unnecessarily flatten a network's broadcast domain and blast radius.
- **Static IP conflicts with the DHCP pool** — manually assigning an address that falls inside the range DHCP is also allowed to lease, causing intermittent, hard-to-diagnose duplicate-address conflicts.
- **No DHCP snooping / rogue server detection** on enterprise switches, leaving the network wide open to the rogue DHCP MITM scenario described above.
- **Incorrect default gateway configuration** — a host with a gateway address that doesn't actually correspond to a router on its local subnet will have working local connectivity but be completely unable to reach anything beyond it.
- **Forgetting DHCP lease expiry implications** — assuming an IP-to-device mapping from an hour ago is still valid for logging or access-control purposes, when the lease may have since been reassigned to a different device.

## Related TryHackMe Rooms in This Series

- [What is Networking?](../../fundamentals/what-is-networking/README.md) — introduces IP addressing that subnetting builds directly on.
- [Extending Your Network](../../fundamentals/extending-your-network/README.md) — the routers and switches that make gateway routing and VLAN-based subnetting physically possible.
- [Networking Concepts](../networking-concepts/README.md) — the OSI/TCP-IP and port/socket vocabulary assumed throughout this guide.
- [Networking Core Protocols](../networking-core-protocols/README.md) — examines the TCP, UDP, and ICMP traffic that actually flows once addressing and routing are configured.

## References

- RFC 2131, Dynamic Host Configuration Protocol — https://www.rfc-editor.org/rfc/rfc2131
- RFC 4632, Classless Inter-domain Routing (CIDR): The Internet Address Assignment and Aggregation Plan — https://www.rfc-editor.org/rfc/rfc4632
- RFC 826, An Ethernet Address Resolution Protocol (ARP) — https://www.rfc-editor.org/rfc/rfc826
- RFC 1918, Address Allocation for Private Internets — https://www.rfc-editor.org/rfc/rfc1918
- Cisco, "What Is DHCP (Dynamic Host Configuration Protocol)?" — https://www.cisco.com/c/en/us/products/switches/what-is-dhcp.html
- IANA, IPv4 Address Space Registry — https://www.iana.org/assignments/ipv4-address-space/ipv4-address-space.xhtml
