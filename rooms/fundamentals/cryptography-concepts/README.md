# Cryptography Concepts

**Platform:** TryHackMe · **Type:** Concept Guide (no personal run captured — see note below)
**Primary domain:** Security Fundamentals

> **Note:** This is a concept guide covering the material this TryHackMe room teaches, written from
> public documentation and reference material. It is not a personal walkthrough — no session evidence
> (screenshots, command output, room-specific answers) is included.

## Overview

Cryptography is the practice of protecting information using mathematical techniques so that only
authorized parties can read it, verify it, or trust its origin. It underpins nearly every other
security control: TLS connections, password storage, VPNs, code signing, and disk encryption are all
applications of a small set of core primitives. This guide covers the building blocks — symmetric and
asymmetric encryption, hashing, digital signatures, and the TLS handshake — using the standards that
define them (NIST FIPS 197, RFC 8446) rather than vendor marketing terms.

## Core Concepts

### Symmetric Encryption

Symmetric encryption uses a single shared key for both encryption and decryption. It is fast and
well-suited to bulk data protection, but requires that both parties already possess the same secret
key, which creates a key-distribution problem.

- **AES (Advanced Encryption Standard)** — standardized in NIST FIPS 197, AES is a block cipher
  operating on 128-bit blocks with key sizes of 128, 192, or 256 bits. AES-256 is the current baseline
  for high-assurance data-at-rest encryption (full-disk encryption, database encryption, VPN tunnels).
- **Modes of operation** — AES itself only encrypts a single block; a mode of operation defines how it
  handles longer messages. **CBC** (Cipher Block Chaining) chains blocks together but provides no
  built-in integrity check. **GCM** (Galois/Counter Mode) is an *authenticated encryption* mode that
  provides confidentiality and integrity in one pass, which is why it is the default in TLS 1.3 and
  most modern protocols.
- **ChaCha20-Poly1305** — a stream-cipher-based authenticated encryption alternative to AES-GCM, used
  particularly on platforms without hardware AES acceleration (many mobile devices).

### Asymmetric (Public-Key) Encryption

Asymmetric encryption uses a mathematically linked key pair: a public key that can be shared freely and
a private key that must stay secret. Data encrypted with the public key can only be decrypted with the
matching private key, which solves the key-distribution problem symmetric crypto has, at the cost of
being computationally much slower.

- **RSA** — based on the difficulty of factoring the product of two large prime numbers. Common key
  sizes are 2048-bit (current minimum recommended by NIST SP 800-57) and 3072/4096-bit for longer-term
  protection. Used for key exchange, digital signatures, and certificate signing.
- **Elliptic Curve Cryptography (ECC)** — based on the algebraic structure of elliptic curves over
  finite fields. ECC (e.g., ECDSA, ECDH using curves like P-256 or Curve25519) achieves equivalent
  security to RSA with much smaller key sizes, which is why it has become the default for TLS key
  exchange and mobile/IoT use cases.
- **Diffie-Hellman (DH) / Elliptic-Curve Diffie-Hellman (ECDH)** — key *agreement* algorithms that let
  two parties derive a shared secret over an insecure channel without ever transmitting the secret
  itself. This is distinct from encrypting data directly with RSA.

In practice, systems rarely use asymmetric crypto to encrypt bulk data because it's slow. Instead, they
use a **hybrid approach**: asymmetric crypto establishes a shared symmetric session key, and that key
does the actual bulk encryption. TLS is the canonical example of this pattern.

### Hashing vs. Encryption

These are frequently confused but serve different purposes:

| | Encryption | Hashing |
|---|---|---|
| Reversible? | Yes, with the correct key | No — one-way by design |
| Purpose | Confidentiality | Integrity / verification |
| Output length | Proportional to input | Fixed length regardless of input size |
| Example algorithms | AES, RSA | SHA-256, SHA-3, bcrypt |

- **SHA-256 / SHA-3** — standardized in NIST FIPS 180-4 and FIPS 202 respectively, these produce a
  fixed-size digest used for file integrity verification, blockchain, and digital signatures. A good
  cryptographic hash function exhibits the **avalanche effect** (a one-bit input change flips roughly
  half the output bits) and **collision resistance** (it's computationally infeasible to find two
  inputs producing the same hash).
- **Password hashing is different from general-purpose hashing.** Fast hashes like SHA-256 are
  deliberately *bad* for password storage because they make brute-forcing cheap. Purpose-built,
  deliberately slow algorithms — **bcrypt**, **scrypt**, and **Argon2** (winner of the Password Hashing
  Competition, and recommended in OWASP's Password Storage Cheat Sheet) — add computational cost and
  salting to resist offline cracking.
- **MD5 and SHA-1 are considered broken** for security purposes: both have practical collision attacks
  published, and NIST deprecated SHA-1 for digital signatures. They may still appear for non-security
  checksums but should never be used for password storage or certificate signing.

### Digital Signatures

A digital signature combines hashing and asymmetric encryption to provide three properties at once:
**integrity** (the message wasn't altered), **authenticity** (it came from the claimed sender), and
**non-repudiation** (the sender can't credibly deny signing it). The process: hash the message, encrypt
the hash with the signer's private key, and attach that as the signature. Anyone with the signer's
public key can decrypt the signature, recompute the hash independently, and compare. This is the basis
for code signing, TLS certificates, and DNSSEC.

### TLS 1.3 Handshake Basics

TLS (Transport Layer Security), defined in RFC 8446 for version 1.3, is what secures HTTPS traffic. A
simplified handshake flow:

1. **ClientHello** — the client proposes supported cipher suites, TLS version, and a random value.
2. **ServerHello** — the server picks a cipher suite, sends its certificate (containing its public key,
   signed by a Certificate Authority), and its own random value.
3. **Key exchange** — both sides use (EC)DHE to derive a shared symmetric session key without ever
   transmitting it directly, providing **forward secrecy** (a compromised long-term private key later
   cannot decrypt past sessions, because the session key was never derivable from it alone).
4. **Certificate verification** — the client validates the server's certificate chain up to a trusted
   root CA, confirming the server is who it claims to be.
5. **Application data** — from this point, all traffic is encrypted with the negotiated symmetric key
   (typically AES-GCM or ChaCha20-Poly1305).

TLS 1.3 removed support for older, weaker ciphers (RC4, static RSA key exchange, CBC-mode ciphers
without authentication) and cut the handshake from two round trips to one, improving both security and
performance over TLS 1.2.

## Why It Matters for Security

Nearly every technical control a security professional touches ultimately depends on one of these
primitives being implemented and configured correctly. A penetration tester checking for weak TLS
configurations, an analyst validating a file hash against a threat intel feed, or an engineer deciding
whether to use bcrypt or SHA-256 for password storage are all applying the same core concepts. Getting
the choice wrong — using a broken algorithm, a fast hash for passwords, or skipping certificate
validation — routinely shows up as a finding in penetration test reports and as root cause in breach
post-mortems.

## Common Pitfalls / Misconfigurations

- **Using SHA-256/MD5 directly for password storage** instead of a slow, salted algorithm like bcrypt
  or Argon2 — makes offline cracking dramatically faster.
- **Hardcoding or reusing encryption keys**, or storing keys alongside the data they protect, defeating
  the purpose of encryption entirely.
- **Disabling certificate validation** ("just to get it working") in application code, which removes
  TLS's authenticity guarantee and opens the door to man-in-the-middle attacks.
- **Using ECB mode** for block ciphers — it encrypts identical plaintext blocks to identical ciphertext
  blocks, leaking structural patterns (the classic "ECB penguin" image example).
- **Rolling your own crypto.** Custom, unreviewed algorithms almost always contain exploitable flaws;
  use vetted, standardized libraries and algorithms instead.
- **Ignoring key rotation and expiry**, leaving long-lived keys exposed to a larger window of potential
  compromise.

## Related TryHackMe Rooms in This Series

- [The CIA Triad](../../easy/the-cia-triad/README.md)
- [Defensive Security Intro](../../easy/defensive-security-intro/README.md)

## References

- NIST FIPS 197, *Advanced Encryption Standard (AES)*: https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.197.pdf
- NIST FIPS 180-4, *Secure Hash Standard (SHS)*: https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.180-4.pdf
- NIST FIPS 202, *SHA-3 Standard*: https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.202.pdf
- NIST SP 800-57 Part 1, *Recommendation for Key Management*: https://csrc.nist.gov/pubs/sp/800/57/pt1/r5/final
- RFC 8446, *The Transport Layer Security (TLS) Protocol Version 1.3*: https://www.rfc-editor.org/rfc/rfc8446
- OWASP Password Storage Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html
- OWASP Cryptographic Storage Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html
