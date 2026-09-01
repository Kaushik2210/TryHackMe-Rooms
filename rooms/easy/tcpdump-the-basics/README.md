# Tcpdump: The Basics

**Platform:** TryHackMe · **Type:** Concept Guide (no personal run captured — see note below)
**Primary domain:** Network Analysis

> **Note:** This is a concept guide covering the material this TryHackMe room teaches, written from
> public documentation and reference material. It is not a personal walkthrough — no session evidence
> (screenshots, command output, room-specific scenario details, or answers) is included. This guide
> covers the underlying professional skill domain the room is built around, not TryHackMe's specific
> scenario content.

## Overview

`tcpdump` is a command-line packet analyzer built on `libpcap`, the same capture library that underlies
Wireshark and `tshark`. It has shipped on virtually every Unix-like system since the late 1980s, which
makes it the tool most likely to already be present on a production server, a jump box, or a minimal
container image when a network problem needs to be diagnosed with no GUI available. Where Wireshark
favors deep, interactive protocol dissection, `tcpdump` favors speed, scriptability, and low overhead —
it is frequently the first tool reached for during an incident, with the resulting `.pcap` handed off to
Wireshark afterward for deeper analysis.

## Core Concepts

### Basic invocation

The minimal `tcpdump` command captures on a chosen interface and prints a one-line summary per packet:

```bash
tcpdump -i eth0
```

Common flags used in almost every invocation:

| Flag | Meaning |
|---|---|
| `-i <iface>` | interface to capture on (`any` captures on all interfaces on Linux) |
| `-n` | do not resolve hostnames (faster, avoids DNS noise in the capture) |
| `-nn` | do not resolve hostnames *or* port names (shows raw port numbers) |
| `-v`, `-vv`, `-vvv` | increasing verbosity of packet output |
| `-c <n>` | stop after capturing `n` packets |
| `-w <file>.pcap` | write raw packets to a file instead of printing them |
| `-r <file>.pcap` | read and print packets from a previously saved capture |
| `-A` | print packet payload in ASCII |
| `-X` | print packet payload in hex and ASCII |
| `-s <n>` | snapshot length — how many bytes of each packet to capture (`-s 0` or a large value captures the full packet) |

A typical diagnostic invocation combines several of these:

```bash
tcpdump -i eth0 -nn -c 100 -w capture.pcap port 80
```

### BPF filter expressions

`tcpdump` filters are written in Berkeley Packet Filter (BPF) syntax — the same filter language used in
Wireshark's *capture filter* field (not its display filter field, which uses different syntax entirely).
A BPF expression is built from **primitives** (qualifiers like `host`, `net`, `port`, `proto`) combined
with boolean operators (`and`/`&&`, `or`/`||`, `not`/`!`).

Common primitives:

```bash
tcpdump -i eth0 host 10.0.0.5          # traffic to or from a specific host
tcpdump -i eth0 src host 10.0.0.5      # traffic only from that host
tcpdump -i eth0 dst host 10.0.0.5      # traffic only to that host
tcpdump -i eth0 net 10.0.0.0/24        # traffic within a subnet
tcpdump -i eth0 port 443               # traffic on a specific port, either direction
tcpdump -i eth0 tcp port 22            # restrict to a protocol as well as a port
tcpdump -i eth0 portrange 1-1024       # a range of ports
```

Combining primitives with boolean logic:

```bash
tcpdump -i eth0 host 10.0.0.5 and port 443
tcpdump -i eth0 src 10.0.0.5 and not dst port 22
tcpdump -i eth0 '(tcp or udp) and port 53'
```

Quoting the expression (single quotes) is good practice once parentheses appear, because otherwise the
shell interprets them before `tcpdump` does.

### Reading TCP flags without a GUI

A distinctive `tcpdump` skill is reading TCP flag combinations directly from the terse output line,
since there is no packet-detail tree to expand. Flags appear abbreviated: `S` (SYN), `.` (ACK), `F`
(FIN), `R` (RST), `P` (PSH). A normal TCP three-way handshake shows as `S` from the client, `S.` from
the server, and `.` back from the client. BPF also supports filtering directly on flag bits, which is
useful for isolating handshake or teardown traffic:

```bash
tcpdump -i eth0 'tcp[tcpflags] == tcp-syn'                 # SYN only (new connection attempts)
tcpdump -i eth0 'tcp[tcpflags] & (tcp-syn|tcp-ack) != 0'    # SYN or SYN-ACK packets
```

### When to reach for tcpdump over Wireshark

- **No GUI available** — headless servers, containers, and minimal cloud images almost always have
  `tcpdump` (or can install it trivially) but rarely have an X server for Wireshark.
- **Remote diagnosis over SSH** — `tcpdump` output can be piped live to a local Wireshark instance for
  GUI analysis without ever writing a file on the remote host:
  ```bash
  ssh user@remote-host "tcpdump -i eth0 -w - 'port 443'" | wireshark -k -i -
  ```
- **Scripting and automation** — because `tcpdump` is a plain CLI tool, its output integrates naturally
  into shell pipelines, cron jobs, and monitoring scripts in a way a GUI tool cannot.
- **Low overhead** — capturing to `-w file.pcap` with a minimal filter is lighter weight than running a
  full GUI dissector against live high-volume traffic, which matters on production systems.

The common professional pattern is "capture narrow with `tcpdump`, analyze deep with Wireshark": use a
tight BPF filter and `-w` to produce a small, targeted `.pcap` on the host where the traffic actually
occurs, then transfer that file to a workstation for full protocol dissection.

## Why It Matters for Security

`tcpdump` is frequently the only capture tool immediately available during an active incident on a
production Linux host, making it a core skill for both incident responders and system administrators.
Because BPF filter syntax also underlies Wireshark's capture filters, iptables/nftables-adjacent tooling,
and various eBPF-based observability tools, learning BPF primitives here transfers directly to other
parts of the security and networking toolchain. On the offensive side, `tcpdump` is commonly used during
internal penetration tests and red team engagements to passively fingerprint network traffic (broadcast
protocols, ARP, DHCP, CDP/LLDP) without generating the noise an active scanner would.

## Common Pitfalls / Misconfigurations

- **Running without `-nn` on a slow or hostile network** causes `tcpdump` to attempt reverse-DNS lookups
  for every address, which can significantly slow capture and, on a hostile network, leak the fact that
  a capture is occurring via visible DNS queries.
- **Forgetting `-s 0` (or a sufficiently large snaplen) when payload matters.** Older or default snaplen
  values can truncate packets, silently discarding the very payload bytes an analyst wants to inspect
  with `-A`/`-X`.
- **Filtering only on `port`, not `src`/`dst`,** when the intent is one-directional — `port 443` matches
  traffic in both directions, so a filter meant to isolate only client-to-server requests needs
  `dst port 443` (or the appropriate `src`/`dst` qualifier).
- **Not quoting BPF expressions with parentheses or `!`**, which the shell may interpret as job control
  or globbing before `tcpdump` sees them, producing a confusing "syntax error" that is actually a shell
  quoting problem.
- **Capturing without a filter on a busy interface** and filling disk or memory before the interesting
  traffic occurs — always scope the filter and/or use `-c`/`-C` (file size rotation) on production
  systems.
- **Running as a non-privileged user.** Raw packet capture requires elevated privileges or the
  `CAP_NET_RAW`/`CAP_NET_ADMIN` capabilities; many "permission denied" issues in the field trace back to
  this rather than a genuine filter problem.

## Related TryHackMe Rooms in This Series

- [`../wireshark-the-basics/README.md`](../wireshark-the-basics/README.md) — the GUI counterpart; BPF
  syntax learned here is reused directly in Wireshark's capture-filter field, and a `tcpdump -w` file
  opens natively in Wireshark for deeper dissection.
- [`../networking-essentials/README.md`](../networking-essentials/README.md) and
  [`../networking-core-protocols/README.md`](../networking-core-protocols/README.md) — TCP/IP
  fundamentals that give BPF filter fields (`tcp[tcpflags]`, port numbers, subnets) their meaning.
- [`../linux-cli-basics/README.md`](../linux-cli-basics/README.md) and
  [`../linux-shells/README.md`](../linux-shells/README.md) — shell quoting and piping conventions used
  when scripting `tcpdump`.
- [`../offensive-security-intro/README.md`](../offensive-security-intro/README.md) — passive network
  reconnaissance as part of a broader assessment methodology.

## References

- [tcpdump.org — official manual page](https://www.tcpdump.org/manpages/tcpdump.1.html)
- [pcap-filter(7) — BPF primitive syntax reference](https://www.tcpdump.org/manpages/pcap-filter.7.html)
- [tcpdump.org — public repository of example filters](https://www.tcpdump.org/)
- [Wireshark Wiki — CaptureFilters (shared BPF syntax)](https://wiki.wireshark.org/CaptureFilters)
