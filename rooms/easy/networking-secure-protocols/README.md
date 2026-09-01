# Networking Secure Protocols

**Platform:** TryHackMe · **Type:** Concept Guide (no personal run captured — see note below)
**Primary domain:** Networking

> **Note:** This is a concept guide covering the material this TryHackMe room teaches, written from
> public documentation and reference material. It is not a personal walkthrough — no session evidence
> (screenshots, command output, room-specific answers) is included.

## Overview

The core protocols covered elsewhere in this series — TCP, UDP, IP — were designed in an era with no expectation of a hostile network, and none of them provide confidentiality, integrity, or authentication on their own. TryHackMe's "Networking Secure Protocols" room covers the protocols built to close that gap: TLS/SSL, which secures point-to-point application traffic (most visibly HTTPS); SSH, which secures remote administration; and VPN technologies (IPsec and WireGuard), which secure entire network paths. Understanding how each actually achieves confidentiality and authentication — and where their historical predecessors failed — is essential for both defending infrastructure and recognising when "encrypted" doesn't actually mean "secure."

## Core Concepts

### SSL and TLS: securing point-to-point traffic

**Transport Layer Security (TLS)** is the modern name for what was originally SSL (Secure Sockets Layer), developed by Netscape in the 1990s. SSL 2.0 and 3.0 are now formally deprecated and prohibited from use (RFC 6176 deprecates SSLv2; RFC 7568 deprecates SSLv3) due to serious, practically exploitable weaknesses. The IETF took over standardisation starting with TLS 1.0 (RFC 2246), through TLS 1.2 (RFC 5246), to the current version, **TLS 1.3** (RFC 8446, published 2018), which is now the recommended baseline for new deployments.

TLS provides three properties: confidentiality (data is encrypted in transit), integrity (tampering is detectable), and authentication (typically the server proves its identity via an X.509 certificate; client certificates are optional and less common). A simplified TLS 1.3 handshake:

1. **ClientHello** — the client proposes supported TLS versions, cipher suites, and (critically, in TLS 1.3) sends its key share for the key exchange immediately, rather than waiting for a round trip.
2. **ServerHello** — the server selects the cipher suite, sends its own key share, and sends its certificate plus a signature proving it holds the certificate's private key.
3. **Key derivation** — both sides independently derive the same shared symmetric session keys using an ephemeral Diffie-Hellman exchange (TLS 1.3 mandates forward secrecy, meaning a compromised long-term key cannot be used to decrypt previously captured traffic).
4. **Finished** — both sides confirm the handshake completed correctly, and the connection switches to encrypted application data.

TLS 1.3's headline improvement over 1.2 is reducing this to effectively one round trip (versus two for TLS 1.2's handshake) and removing support for a long list of legacy, weak cryptographic algorithms (like RC4, static RSA key exchange, and CBC-mode ciphers vulnerable to padding oracle attacks) entirely, rather than merely discouraging them.

### SSH: securing remote administration

**Secure Shell (SSH)** replaced Telnet and rlogin — protocols that transmitted credentials and session data in plaintext — as the standard for remote command-line access, defined across RFC 4251 (architecture), RFC 4252 (authentication), RFC 4253 (transport layer), and RFC 4254 (connection protocol), and conventionally running over TCP port 22. SSH's architecture layers three protocols: the transport layer negotiates the server's identity (via a host key) and establishes an encrypted, integrity-protected channel; the authentication layer verifies the client (via password, public key, or other mechanisms); and the connection layer multiplexes the encrypted channel into logical channels (a shell session, port forwards, SFTP transfers) over the single underlying connection. Public-key authentication is generally preferred over passwords because it isn't vulnerable to credential brute-forcing or reuse from breached password lists in the same way, and it can be paired with a passphrase-protected private key for defence in depth.

### VPNs: securing an entire network path

A **Virtual Private Network (VPN)** extends a private network across a public one by tunnelling traffic through an encrypted channel, making a remote host (or network) appear as if it's directly attached to the private network.

**IPsec** (Internet Protocol Security), defined by the architecture in RFC 4301, operates at the network layer and is composed of several sub-protocols: **AH (Authentication Header)** provides integrity and authentication (but not confidentiality); **ESP (Encapsulating Security Payload)** provides confidentiality plus optional integrity/authentication and is the far more commonly deployed of the two; and **IKE (Internet Key Exchange)**, now at version 2 (IKEv2, RFC 7296), automates negotiating and refreshing the cryptographic keys used to protect traffic. IPsec can run in **transport mode** (encrypting only the payload of each IP packet, used for host-to-host protection) or **tunnel mode** (encapsulating the entire original IP packet inside a new one, used for site-to-site or remote-access VPNs) — the latter is what most commercial and enterprise VPN products actually use.

**WireGuard** is a comparatively recent VPN protocol designed for simplicity and a minimal audit surface, built on the Noise Protocol Framework for its cryptographic handshake and using a fixed, modern, non-negotiable cipher suite (ChaCha20 for encryption, Poly1305 for authentication, Curve25519 for key exchange) rather than the large negotiable suite IPsec/TLS support. Its design philosophy — a small, auditable codebase with no cryptographic agility to misconfigure — trades IPsec's flexibility and legacy compatibility for a dramatically smaller attack surface, and it has since been merged directly into the Linux kernel.

### What "secure" actually depends on

All of these protocols share a structural pattern worth internalising: an unauthenticated key exchange (or a certificate-based one) establishes a shared secret, that secret derives symmetric session keys (because symmetric cryptography is far cheaper than asymmetric for bulk data), and every subsequent message is protected using those session keys until the session ends. Security in every case rests on that initial exchange being both confidential and correctly authenticated — an attacker who can interpose during the handshake (a man-in-the-middle) and get their own key accepted as legitimate defeats the encryption regardless of how strong the cipher is afterward.

## Why It Matters for Security

- **TLS downgrade attacks** (such as the historical POODLE attack against SSLv3, or forcing a fallback to weaker cipher suites) exploit clients or servers that still permit deprecated protocol versions for "compatibility," which is exactly why disabling SSLv2/SSLv3/TLS 1.0/1.1 is a standard hardening step.
- **Certificate validation failures** — an application that accepts any certificate (self-signed, expired, hostname-mismatched) without proper validation defeats TLS's server-authentication guarantee entirely, opening the door to trivial MITM interception even though the traffic is technically "encrypted."
- **SSH host key verification is what actually prevents MITM on first connection** — the familiar "authenticity of host can't be established" warning exists precisely because trust-on-first-use has no way to distinguish a legitimate server from an impersonator unless the fingerprint is verified out-of-band.
- **SSH and VPN brute-forcing** remain common attack paths specifically because they're exposed, authenticated entry points to otherwise-protected networks — this is why key-based auth, fail2ban-style rate limiting, and restricting exposure (bastion hosts, VPN-gated access) are standard controls.
- **VPN split-tunnelling misconfiguration** can let a compromised remote endpoint act as a bridge between an untrusted network and the "secure" internal network the VPN was meant to protect.

## Common Pitfalls / Misconfigurations

- **Leaving legacy TLS versions or weak cipher suites enabled** on a server "just in case," which both weakens security and is flagged by essentially every modern vulnerability scanner.
- **Self-signed certificates in production** trained users (or applications) to click through certificate warnings, eroding the entire trust model TLS relies on.
- **Password-only SSH authentication exposed to the internet**, an extremely common target for credential-stuffing and brute-force botnets scanning port 22 continuously.
- **Reusing SSH host keys** across cloned VM images or containers, which breaks the assumption that a host key uniquely identifies one server.
- **Overly broad IPsec/VPN split-tunnel or firewall rules** that grant a remote client full internal network reachability rather than access scoped to what the connection actually needs.
- **Treating "VPN" as synonymous with "secure"** without verifying which protocol, cipher suite, and authentication mechanism the VPN product is actually using underneath its marketing name.

## Related TryHackMe Rooms in This Series

- [Networking Core Protocols](../networking-core-protocols/README.md) — the TCP foundation that TLS and SSH are layered on top of.
- [Networking Concepts](../networking-concepts/README.md) — the OSI layering that clarifies where TLS (often described as Layer 6, though it spans Session/Presentation in practice) sits relative to TCP and application protocols.
- [Networking Essentials](../networking-essentials/README.md) — the addressing and gateway routing that VPN tunnels operate over.
- [What is Networking?](../../fundamentals/what-is-networking/README.md) — the addressing and protocol fundamentals assumed throughout this guide.

## References

- RFC 8446, The Transport Layer Security (TLS) Protocol Version 1.3 — https://www.rfc-editor.org/rfc/rfc8446
- RFC 5246, The Transport Layer Security (TLS) Protocol Version 1.2 — https://www.rfc-editor.org/rfc/rfc5246
- RFC 7568, Deprecating Secure Sockets Layer Version 3.0 — https://www.rfc-editor.org/rfc/rfc7568
- RFC 6176, Prohibiting Secure Sockets Layer (SSL) Version 2.0 — https://www.rfc-editor.org/rfc/rfc6176
- RFC 4251, The Secure Shell (SSH) Protocol Architecture — https://www.rfc-editor.org/rfc/rfc4251
- RFC 4253, The Secure Shell (SSH) Transport Layer Protocol — https://www.rfc-editor.org/rfc/rfc4253
- RFC 4301, Security Architecture for the Internet Protocol (IPsec) — https://www.rfc-editor.org/rfc/rfc4301
- RFC 7296, Internet Key Exchange Protocol Version 2 (IKEv2) — https://www.rfc-editor.org/rfc/rfc7296
- WireGuard, "Protocol & Cryptography" whitepaper — https://www.wireguard.com/papers/wireguard.pdf
- WireGuard official site — https://www.wireguard.com/
- MDN Web Docs, "Transport Layer Security (TLS)" — https://developer.mozilla.org/en-US/docs/Glossary/TLS
