# The CIA Triad

**Platform:** TryHackMe · **Type:** Concept Guide (no personal run captured — see note below)
**Primary domain:** Security Fundamentals

> **Note:** This is a concept guide covering the material this TryHackMe room teaches, written from
> public documentation and reference material. It is not a personal walkthrough — no session evidence
> (screenshots, command output, room-specific answers) is included.

## Overview

The CIA Triad — Confidentiality, Integrity, and Availability — is the foundational model used to reason
about what "security" actually means for a system or piece of data. Nearly every control, policy, and
risk decision in information security maps back to protecting one or more of these three properties.
NIST formally defines the triad in FIPS 199 and expands on it throughout the SP 800-series, where it
is used as the basis for categorizing information systems by potential impact. Understanding the triad
is less about memorizing three words and more about learning to ask, for any asset: what happens if
this is disclosed, altered, or made unavailable — and to whom does that matter?

## Core Concepts

### Confidentiality

Confidentiality means that information is only accessible to authorized individuals, processes, or
systems. NIST SP 800-53 defines it as "preserving authorized restrictions on information access and
disclosure, including means for protecting personal privacy and proprietary information." Concrete
controls that enforce confidentiality include:

- **Encryption at rest and in transit** — AES-256 (FIPS 197) for stored data, TLS 1.3 (RFC 8446) for
  data moving across a network.
- **Access control models** — discretionary access control (DAC), mandatory access control (MAC), and
  role-based access control (RBAC), all covered in NIST SP 800-162 for attribute-based variants.
- **Authentication and least privilege** — multi-factor authentication (MFA) and the principle of least
  privilege (NIST SP 800-53 control AC-6), which limits what a compromised account or process can reach.
- **Data classification and labeling** — tagging data as public, internal, confidential, or restricted
  so that handling controls scale with sensitivity.

The classic failure mode is over-permissioning: a database, S3 bucket, or SMB share configured with
broader read access than the business need requires. This is consistently one of the top root causes
in breach reports (see Verizon's annual DBIR) because it converts a single credential leak or
misconfiguration into a full data exposure.

### Integrity

Integrity ensures that data has not been altered or destroyed in an unauthorized manner, and that it
remains accurate and trustworthy across its lifecycle. NIST SP 800-53 frames this as "guarding against
improper information modification or destruction, and includes ensuring information non-repudiation and
authenticity." Controls here include:

- **Cryptographic hashing** — SHA-256 or SHA-3 (FIPS 180-4 / FIPS 202) to produce a fixed-length digest
  that changes completely if even one bit of the input changes. Hashes are used to verify file
  downloads, detect tampering, and underpin digital signatures.
- **Digital signatures** — RSA or ECDSA signatures that bind a hash to a private key, proving both
  integrity and authenticity (who signed it) and providing non-repudiation.
- **Checksums and parity** — lower-assurance mechanisms (CRC32, TCP checksums) used mainly to catch
  accidental corruption rather than deliberate tampering.
- **Version control and audit logging** — change tracking (git, database transaction logs, SIEM audit
  trails) so that unauthorized modifications can be detected and attributed after the fact.
- **File integrity monitoring (FIM)** — tools like Tripwire or OSSEC that baseline critical system files
  and alert on unexpected changes, a common detection control against rootkits and web shells.

A key distinction worth internalizing: hashing verifies integrity but says nothing about
confidentiality (a hash is not encryption and cannot be reversed to recover the original data), while
encryption protects confidentiality but does not, by itself, guarantee integrity — which is why modern
protocols use authenticated encryption (AES-GCM) or encrypt-then-MAC constructions to get both.

### Availability

Availability ensures that systems and data are accessible to authorized users when needed. NIST SP
800-53 defines it as "ensuring timely and reliable access to and use of information." Controls include:

- **Redundancy** — RAID arrays, redundant power/network paths, clustered application servers, and
  multi-region cloud deployments that eliminate single points of failure.
- **Backups and disaster recovery (DR)** — the 3-2-1 backup rule (three copies, two media types, one
  off-site) plus documented recovery time objectives (RTO) and recovery point objectives (RPO).
- **DDoS mitigation** — rate limiting, traffic scrubbing services, and content delivery networks (CDNs)
  that absorb or filter volumetric attacks before they reach origin infrastructure.
- **Capacity planning and patching cadence** — ensuring systems are resourced for expected load and
  patched promptly, since both resource exhaustion and unpatched crash bugs threaten uptime.

Ransomware is the clearest modern illustration of an availability attack: the attacker does not need to
exfiltrate anything to cause damage — encrypting production data and demanding payment for the key
denies access just as effectively as taking a server offline.

### Tradeoffs Between the Three

The three properties frequently pull against each other, and most real security decisions are
tradeoff decisions rather than pure wins:

| Tradeoff | Example |
|---|---|
| Confidentiality vs. Availability | Strict MFA and short session timeouts reduce unauthorized access but can lock out legitimate users during outages or lost devices. |
| Integrity vs. Availability | Taking a system offline to apply an emergency patch protects integrity but temporarily removes availability. |
| Confidentiality vs. Usability | End-to-end encryption protects data but can block legitimate monitoring, DLP scanning, or legal discovery. |

Security architects use risk assessment (NIST SP 800-30) to decide which property to prioritize for a
given asset — a hospital's patient monitoring system will usually prioritize availability, while a
system storing SSNs will usually prioritize confidentiality.

## Why It Matters for Security

Every vulnerability, control, and incident can be described in terms of which leg of the triad it
threatens. A SQL injection that dumps a customer table is a confidentiality failure. A supply-chain
attack that plants a backdoor in a software update is an integrity failure. A botnet-driven DDoS
against an e-commerce site during a sale is an availability failure. Framing incidents this way is
what lets defenders prioritize: a confidentiality breach of public marketing content is low severity,
while an integrity breach of a firmware update mechanism is catastrophic regardless of how "small" the
change looks. Risk frameworks like NIST SP 800-30 and FAIR both use CIA impact as an input to severity
scoring, and it directly informs incident classification tiers in most SOC playbooks.

## Common Pitfalls / Misconfigurations

- **Treating "encrypted" as synonymous with "secure."** Encryption addresses confidentiality only; a
  system can be fully encrypted and still be a total integrity or availability failure.
- **Ignoring availability in security reviews.** Security teams often over-index on confidentiality and
  under-invest in redundancy and DR testing, leaving availability as the weakest leg.
- **Weak or missing integrity checks on downloaded artifacts.** Skipping signature or hash verification
  on software packages, container images, or firmware opens the door to supply-chain tampering.
- **Overbroad access grants "just in case."** Violates least privilege and expands the confidentiality
  blast radius of any single compromised credential.
- **No tested backups.** Backups that exist but have never been restored in a drill are a common cause
  of failed recovery during real ransomware incidents.

## Related TryHackMe Rooms in This Series

- [Defensive Security Intro](../defensive-security-intro/README.md)
- [Offensive Security Intro](../offensive-security-intro/README.md)
- [Cryptography Concepts](../../fundamentals/cryptography-concepts/README.md)

## References

- NIST FIPS 199, *Standards for Security Categorization of Federal Information and Information Systems*: https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.199.pdf
- NIST SP 800-53 Rev. 5, *Security and Privacy Controls for Information Systems and Organizations*: https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final
- NIST SP 800-30 Rev. 1, *Guide for Conducting Risk Assessments*: https://csrc.nist.gov/pubs/sp/800/30/r1/final
- NIST FIPS 197, *Advanced Encryption Standard (AES)*: https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.197.pdf
- NIST FIPS 180-4, *Secure Hash Standard (SHS)*: https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.180-4.pdf
- Verizon, *Data Breach Investigations Report*: https://www.verizon.com/business/resources/reports/dbir/
