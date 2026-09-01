# DNS in Detail

**Platform:** TryHackMe · **Type:** Concept Guide (no personal run captured — see note below)
**Primary domain:** Web Fundamentals

> **Note:** This is a concept guide covering the material this TryHackMe room teaches, written from
> public documentation and reference material. It is not a personal walkthrough — no session evidence
> (screenshots, command output, room-specific answers) is included.

## Overview

The Domain Name System (DNS) is the distributed lookup service that maps human-readable names like
`example.com` to the machine-readable identifiers — IP addresses, mail server hostnames, and other
records — that networked applications actually need to communicate. It was formally specified in 1987 in
[RFC 1034](https://www.rfc-editor.org/rfc/rfc1034) (concepts and facilities) and
[RFC 1035](https://www.rfc-editor.org/rfc/rfc1035) (implementation and wire format), and it has been
extended many times since without breaking its core design: a hierarchical, delegated, heavily cached
namespace. Because DNS sits in front of almost every other protocol on the internet, it is also one of
the most consequential attack surfaces in networking — controlling or poisoning a DNS answer means
controlling where a client goes next.

## Core Concepts

### The namespace and the hierarchy

DNS names are read right-to-left in terms of authority. The root zone (represented by a trailing dot,
e.g. `example.com.`) is served by 13 logical root server clusters. Below the root sit
**Top-Level Domains** (TLDs) such as `.com`, `.org`, and country-code TLDs like `.uk`. Below a TLD sit
**second-level domains** that organizations register (`example.com`), and below those, any number of
**subdomains** (`www.example.com`, `mail.example.com`). Each level of this tree can be **delegated** to a
different set of name servers, which is what allows DNS to scale to billions of names without any single
server holding the entire dataset. This structure is defined in
[RFC 1034 §3](https://www.rfc-editor.org/rfc/rfc1034).

### Zones, zone files, and authoritative servers

A **zone** is a portion of the namespace that a particular organization administers and serves from its
own name servers. A zone's data is described declaratively in a **zone file**, made up of **resource
records (RRs)**. Every zone has an **SOA (Start of Authority)** record identifying the primary name
server, the responsible party, and timing parameters (refresh, retry, expire, and minimum/negative-cache
TTL) that govern how secondary servers resynchronize. A server that holds a zone's actual data and answers
queries for it directly, without recursing, is called an **authoritative name server** for that zone
([RFC 1035 §3.7](https://www.rfc-editor.org/rfc/rfc1035)).

### Common resource record types

| Type | Purpose |
|------|---------|
| **A** | Maps a name to an IPv4 address ([RFC 1035](https://www.rfc-editor.org/rfc/rfc1035)) |
| **AAAA** | Maps a name to an IPv6 address ([RFC 3596](https://www.rfc-editor.org/rfc/rfc3596)) |
| **CNAME** | An alias: points one name at another name, which is then resolved again |
| **NS** | Delegates a subtree of the namespace to a set of authoritative name servers |
| **MX** | Identifies mail servers for a domain, with a priority/preference value |
| **TXT** | Arbitrary text; widely repurposed for domain verification, SPF, DKIM, and DMARC policy |
| **SOA** | Zone metadata: primary server, admin contact, serial number, and timing/TTL values |
| **PTR** | Reverse mapping, IP address to name, used for reverse DNS lookups |
| **SRV** | Generalized service location record (host + port) defined in [RFC 2782](https://www.rfc-editor.org/rfc/rfc2782) |

Every record also carries a **Time To Live (TTL)**, in seconds, that tells resolvers how long they may
cache the answer before re-querying — a core mechanism for both DNS's scalability and its window for
serving stale or poisoned data.

### The resolution flow: stub, recursive, root, TLD, authoritative

When an application asks to resolve `www.example.com`, the request typically flows through several
distinct actors:

1. **Stub resolver** — a small piece of OS-level client code that has no caching or iterative-query logic
   of its own; it simply forwards the question to a configured recursive resolver.
2. **Recursive resolver** — (an ISP resolver, or a public one such as Cloudflare's `1.1.1.1` or Google's
   `8.8.8.8`) does the actual work on the client's behalf. It first checks its own cache; on a miss, it
   performs **iterative** queries on the client's behalf, walking down the hierarchy.
3. **Root server** — asked "who is authoritative for `.com`?", returns a referral to the `.com` TLD name
   servers (it does not know the final answer itself).
4. **TLD server** — asked the same question about `example.com`, returns a referral to `example.com`'s
   authoritative name servers.
5. **Authoritative name server** — finally returns the actual A/AAAA record (or a CNAME to be chased
   further) for `www.example.com`.

The recursive resolver caches the final answer, and every referral along the way, according to each
record's TTL, so that subsequent queries for the same or related names can be answered without repeating
the full walk. This iterative-vs-recursive query distinction is described in
[RFC 1034 §4.3](https://www.rfc-editor.org/rfc/rfc1034).

### Transport: UDP, TCP, and encrypted DNS

Classic DNS queries run over **UDP port 53** because most queries and responses are small and a full
TCP handshake would be wasteful overhead for a lookup that may happen thousands of times a second across
a network. DNS falls back to **TCP port 53** when a response is too large for a single UDP datagram (or
when EDNS0, [RFC 6891](https://www.rfc-editor.org/rfc/rfc6891), negotiates a larger UDP payload instead)
and for zone transfers between primary and secondary servers. Because plaintext UDP/TCP DNS is trivially
observable and spoofable on the wire, two encrypted transports have since been standardized: **DNS over
TLS (DoT)**, on port 853 ([RFC 7858](https://www.rfc-editor.org/rfc/rfc7858)), and **DNS over HTTPS
(DoH)** ([RFC 8484](https://www.rfc-editor.org/rfc/rfc8484)), which tunnels DNS queries inside ordinary
HTTPS traffic so it is far harder for a network observer to distinguish or block.

### The message format, at a glance

Every DNS message — query or response — shares one wire format: a fixed 12-byte **header** (containing a
transaction ID, flags such as QR/opcode/RCODE, and four count fields), followed by four variable-length
sections: **Question** (the name/type/class being asked about), **Answer** (RRs that directly answer the
question), **Authority** (NS records for the zone that is authoritative), and **Additional** (extra
helpful records, such as the IP address that goes with an NS record — "glue" records). The full binary
layout is specified in [RFC 1035 §4](https://www.rfc-editor.org/rfc/rfc1035).

## Why It Matters for Security

DNS's position as the very first step in nearly every network interaction makes it a high-value target:

- **Cache poisoning / spoofing** — because classic DNS over UDP has no built-in response authentication
  beyond a 16-bit transaction ID and matching source port, an attacker who can guess or race a legitimate
  response can inject a forged answer into a resolver's cache, redirecting every subsequent client that
  queries that resolver until the TTL expires. **DNSSEC** ([RFC 4033](https://www.rfc-editor.org/rfc/rfc4033)–[4035](https://www.rfc-editor.org/rfc/rfc4035))
  addresses this by cryptographically signing zone data so resolvers can validate authenticity.
- **Subdomain takeover** — when a DNS record (commonly a CNAME) still points at a third-party service
  (a cloud storage bucket, a SaaS app, a PaaS deployment) that has since been deprovisioned, an attacker
  who claims that same resource on the third-party service effectively takes control of the subdomain,
  inheriting its trust and any cookies/CORS trust scoped to it.
- **DNS tunneling** — because DNS queries are almost universally allowed to leave a network (even a
  tightly firewalled one), attackers encode arbitrary data into subdomain labels or TXT record responses
  to exfiltrate data or maintain command-and-control channels that blend in with legitimate lookup
  traffic.
- **Domain generation algorithms (DGAs)** — malware families that algorithmically generate large numbers
  of candidate C2 domains use DNS's open, low-friction registration model to make takedown difficult;
  defenders often look for the resulting burst of NXDOMAIN responses as a detection signal.
- **Open resolvers / amplification** — misconfigured recursive resolvers that answer queries from any
  source IP can be abused as reflectors in DNS amplification DDoS attacks, since a small spoofed query
  can trigger a much larger response directed at a victim.

## Common Pitfalls / Misconfigurations

- **Overly permissive zone transfers (AXFR)** — a name server that allows unauthenticated AXFR requests
  from arbitrary hosts leaks the entire zone file, including internal hostnames that were never meant to
  be public, to anyone who asks.
- **Dangling DNS records** — CNAMEs, MX records, or NS delegations left pointing at decommissioned
  infrastructure are a leading cause of subdomain takeover and can persist for years unnoticed.
- **Overly long TTLs on records that may need to change quickly** — during an incident (e.g. rotating off
  a compromised IP) a long TTL means clients and caching resolvers keep using the old, bad answer well
  past when the operator has already fixed it.
- **No DNSSEC validation** — without DNSSEC, a resolver has no cryptographic way to tell a legitimate
  answer from a forged one; validation must be explicitly enabled and the zone must be signed for the
  protection to exist end-to-end.
- **Treating internal DNS names as a security boundary** — an internal-only hostname is not a secret and
  is not authentication; assuming that "attackers can't guess this subdomain" provides real protection is
  a form of security-by-obscurity that fails against enumeration and certificate-transparency log
  scraping.

## Related TryHackMe Rooms in This Series

- [HTTP in Detail](../http-in-detail/README.md) — the protocol that runs immediately after DNS
  resolution completes, once the client has an IP address to connect to.
- [How Websites Work](../how-websites-work/README.md) — places DNS resolution as Step 1 in the broader
  end-to-end pipeline of loading a web page.
- [Putting it all together](../putting-it-all-together/README.md) — ties DNS, transport, and application
  layers together in a single narrative.

## References

- [RFC 1034 — Domain Names: Concepts and Facilities](https://www.rfc-editor.org/rfc/rfc1034)
- [RFC 1035 — Domain Names: Implementation and Specification](https://www.rfc-editor.org/rfc/rfc1035)
- [RFC 2782 — A DNS RR for specifying the location of services (SRV)](https://www.rfc-editor.org/rfc/rfc2782)
- [RFC 3596 — DNS Extensions to Support IP Version 6](https://www.rfc-editor.org/rfc/rfc3596)
- [RFC 4033 — DNS Security Introduction and Requirements](https://www.rfc-editor.org/rfc/rfc4033)
- [RFC 4035 — Protocol Modifications for the DNS Security Extensions](https://www.rfc-editor.org/rfc/rfc4035)
- [RFC 6891 — Extension Mechanisms for DNS (EDNS0)](https://www.rfc-editor.org/rfc/rfc6891)
- [RFC 7858 — Specification for DNS over Transport Layer Security (DoT)](https://www.rfc-editor.org/rfc/rfc7858)
- [RFC 8484 — DNS Queries over HTTPS (DoH)](https://www.rfc-editor.org/rfc/rfc8484)
- [ICANN — Root Server System](https://www.icann.org/en/system/files/files/root-server-system-04mar19-en.pdf)
- [MDN — DNS](https://developer.mozilla.org/en-US/docs/Glossary/DNS)
