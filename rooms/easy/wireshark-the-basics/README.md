# Wireshark: The Basics

**Platform:** TryHackMe · **Type:** Concept Guide (no personal run captured — see note below)
**Primary domain:** Network Analysis

> **Note:** This is a concept guide covering the material this TryHackMe room teaches, written from
> public documentation and reference material. It is not a personal walkthrough — no session evidence
> (screenshots, command output, room-specific scenario details, or answers) is included. This guide
> covers the underlying professional skill domain the room is built around, not TryHackMe's specific
> scenario content.

## Overview

Wireshark is the de facto standard open-source packet analyzer, built on top of the same `libpcap`/`Npcap`
capture libraries that power `tcpdump` and most other traffic-capture tooling. Where `tcpdump` is a
command-line capture-and-filter tool, Wireshark adds a graphical dissector engine that decodes hundreds
of protocols down to individual fields, letting an analyst inspect a packet's contents layer by layer
rather than as a raw hex dump. It is used daily by network engineers for troubleshooting, by SOC
analysts for traffic-based threat hunting, and by penetration testers for protocol analysis during
internal engagements. This guide covers the mental model and filter syntax that make Wireshark useful,
independent of any specific captured file.

## Core Concepts

### Capture filters vs. display filters

Wireshark exposes two distinct filtering languages that are frequently confused by newcomers:

- **Capture filters** use Berkeley Packet Filter (BPF) syntax — the same syntax `tcpdump` uses — and are
  applied *before* a packet is written to the capture buffer. A capture filter is set in the "Capture
  Options" dialog and cannot be changed after capture has started; anything it excludes is gone
  permanently. Example: `host 192.168.1.10 and port 443`.
- **Display filters** use Wireshark's own field-based syntax and are applied *after* capture, purely to
  the view. They can be changed, combined, and removed at any time without losing data, which makes them
  the far more common tool for day-to-day analysis. Example: `ip.addr == 192.168.1.10 && tcp.port == 443`.

The general rule taught by the Wireshark project itself: capture broadly (or not at all, if disk/traffic
volume allows) and filter narrowly on display, because a display filter can be revised as the
investigation evolves and a capture filter cannot.

### Display filter syntax fundamentals

Display filters are expressions built from protocol/field names, comparison operators, and logical
operators:

| Operator | Meaning | Example |
|---|---|---|
| `==` / `eq` | equals | `ip.addr == 10.0.0.5` |
| `!=` / `ne` | not equal | `tcp.port != 80` |
| `&&` / `and`, `\|\|` / `or`, `!` / `not` | logical combination | `http and tcp.port == 80` |
| `contains` | substring match | `http.request.uri contains "login"` |
| `matches` | regex match | `http.host matches "(?i)admin"` |

Commonly used field-based filters an analyst reaches for constantly:

```text
http.request.method == "POST"
http.response.code == 404
tcp.port == 443
tcp.flags.syn == 1 && tcp.flags.ack == 0
dns.qry.name contains "evil"
tls.handshake.type == 1
ip.src == 192.168.1.0/24
frame contains "password"
```

Filters can be typed directly into the filter bar, or built by right-clicking any field in the packet
detail pane and choosing "Apply as Filter" — a fast way to learn the field names without memorizing the
protocol dictionary. A green filter bar means valid syntax; red means the filter is malformed and is
not being applied.

### The three-pane interface and the OSI model

Wireshark's main window is split into three panes that map directly onto the layered structure of a
network frame:

1. **Packet list pane** — one row per captured frame, showing summary columns (time, source,
   destination, protocol, length, info).
2. **Packet details pane** — the selected frame decoded into an expandable tree, generally ordered from
   the outermost layer inward: Frame → Ethernet (Layer 2) → IP (Layer 3) → TCP/UDP (Layer 4) →
   application protocol such as HTTP, DNS, or TLS (Layer 5–7). Each tree node can be expanded to show
   individual header fields (flags, sequence numbers, TTL, checksums, and so on).
3. **Packet bytes pane** — the raw hex and ASCII representation of the frame; selecting a field in the
   details pane highlights the corresponding bytes here, which is how you learn to read a header by hand
   instead of only trusting the dissector.

Understanding that the details pane is literally the OSI/TCP-IP stack unwound top-to-bottom is the key
insight for reading unfamiliar traffic: if TCP looks fine but the application layer is missing or
malformed, the problem is above Layer 4; if TCP itself shows retransmissions or resets, the problem is
at or below Layer 4.

### Following a stream

Right-clicking any packet and choosing "Follow → TCP Stream" (or UDP/HTTP/TLS Stream) reassembles all
the packets belonging to that one conversation into a single readable transcript, color-coded by
direction. This is the primary technique for reading an HTTP request/response pair, a plaintext
protocol exchange, or a full file transfer as a human-readable whole instead of packet-by-packet.

### Coloring rules and statistics

Wireshark ships with default coloring rules (e.g., black-on-red for TCP resets and errors, black-on-yellow
for retransmissions, light-purple for TCP traffic in general) that let an analyst spot anomalies visually
while scrolling. The **Statistics** menu — particularly *Protocol Hierarchy*, *Conversations*, and
*Endpoints* — gives a quantitative summary of a capture (which protocols dominate, which hosts talk the
most, which flows are largest) before diving into individual packets, which is the recommended starting
point for any unfamiliar `.pcap`.

## Why It Matters for Security

Packet capture analysis sits at the center of both blue-team and red-team work. For defenders, Wireshark
(or its headless sibling `tshark`) is used to validate IDS/IPS alerts against ground-truth traffic,
confirm data exfiltration over unusual ports or protocols, and reconstruct exactly what an attacker sent
during a confirmed intrusion — often the only way to distinguish a false positive from a real compromise.
For offensive practitioners, packet analysis is used to understand application protocols during
assessments, verify whether traffic is encrypted as claimed, and troubleshoot C2 or tooling issues during
an engagement. Because Wireshark decodes so many protocols natively, it is also a fast way to learn how a
protocol actually behaves on the wire rather than how its RFC describes it.

## Common Pitfalls / Misconfigurations

- **Confusing capture and display filter syntax.** Typing a display-filter expression like
  `tcp.port == 443` into the capture-filter box (or vice versa with BPF syntax like `port 443`) produces
  a syntax error or silently captures everything — the two languages are not interchangeable.
- **Capturing on the wrong interface.** On a multi-homed host or VM, capturing on `eth0` when the traffic
  of interest actually traverses a VPN or virtual adapter yields an empty or irrelevant capture.
- **Not using promiscuous mode when required**, or capturing on a switched network segment without port
  mirroring/SPAN configured, resulting in only seeing traffic addressed to the local interface rather
  than the broader segment.
- **Over-trusting the "Protocol" column.** Wireshark's dissector guesses protocol by port and payload
  heuristics; traffic deliberately run on a non-standard port (e.g., HTTP on 8443) can be mis-labeled,
  and analysts should verify with "Decode As" rather than assuming the summary column is authoritative.
- **Applying an overly narrow display filter too early** and missing surrounding context — e.g. filtering
  straight to `http` and missing the DNS resolution or TCP handshake anomalies that explain *why* the
  HTTP request looks the way it does.
- **Forgetting that TLS-encrypted traffic hides application content by default.** Without a
  pre-shared session key log (`SSLKEYLOGFILE`) or a private key configured in Wireshark's TLS
  preferences, HTTPS/TLS payloads remain opaque regardless of filter syntax.

## Related TryHackMe Rooms in This Series

- [`../tcpdump-the-basics/README.md`](../tcpdump-the-basics/README.md) — the command-line counterpart;
  BPF capture-filter syntax learned there is directly reusable in Wireshark's capture-filter field.
- [`../networking-essentials/README.md`](../networking-essentials/README.md) and
  [`../networking-core-protocols/README.md`](../networking-core-protocols/README.md) — the protocol
  fundamentals (TCP handshake, DNS, HTTP) that give the packet detail pane meaning.
- [`../defensive-security-intro/README.md`](../defensive-security-intro/README.md) — where packet
  analysis fits into the broader SOC/defensive workflow.
- [`../junior-security-analyst-intro/README.md`](../junior-security-analyst-intro/README.md) — packet
  capture as one input among several (SIEM, EDR, firewall logs) a Tier 1 analyst triages.

## References

- [Wireshark User's Guide](https://www.wireshark.org/docs/wsug_html_chunked/)
- [Wireshark Display Filter Reference](https://www.wireshark.org/docs/dfref/)
- [Wireshark Wiki — CaptureFilters](https://wiki.wireshark.org/CaptureFilters)
- [Wireshark Wiki — DisplayFilters](https://wiki.wireshark.org/DisplayFilters)
- [Wireshark Wiki — Following TCP Streams](https://wiki.wireshark.org/Following_TCP_Streams)
- [tcpdump/libpcap — pcap-filter(7) man page](https://www.tcpdump.org/manpages/pcap-filter.7.html)
