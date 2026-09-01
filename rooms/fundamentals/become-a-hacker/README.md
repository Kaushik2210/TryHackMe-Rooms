# Become a Hacker

**Platform:** TryHackMe · **Type:** Concept Guide (no personal run captured — see note below)
**Primary domain:** Security Fundamentals

> **Note:** This is a concept guide covering the material this TryHackMe room teaches, written from
> public documentation and reference material. It is not a personal walkthrough — no session evidence
> (screenshots, command output, room-specific answers) is included.

## Overview

"Become a Hacker" is TryHackMe's orientation room for the offensive security learning path: it maps
out what an ethical hacker (penetration tester / red teamer) actually needs to learn, in what order,
and why. Offensive security is not a single skill but a stack of dependent competencies — you cannot
usefully exploit a web application without understanding HTTP and TCP/IP, and you cannot pivot through
a Windows domain without understanding how Active Directory authentication works in the first place.
This guide lays out that skill tree as a concrete, tool-and-certification-referenced roadmap rather than
a vague "learn to hack" pitch.

## Core Concepts

### The Offensive Skill Tree

Ethical hacking skills build on each other in a fairly strict dependency order. Skipping a layer usually
just means re-learning it later while also trying to learn the layer above it.

**1. Networking fundamentals.** TCP/IP, the OSI model, subnetting, DNS, routing, and common protocols
(HTTP/S, SMB, FTP, SSH, RDP) are the substrate everything else runs on. Every exploit ultimately travels
over a network, and every attacker needs to be able to read a packet capture and reason about what a
port scan result implies about the service behind it.

**2. Linux and Windows fundamentals.** Offensive tooling runs overwhelmingly on Linux (most distributions
used for this are Debian-based, commonly Kali Linux or Parrot OS), but the majority of enterprise targets
run Windows. Comfort with the Linux shell (permissions, processes, package management, cron) and with
Windows internals (the registry, services, PowerShell, NTFS permissions) is a prerequisite for everything
that follows — you cannot write an exploit for an OS you cannot navigate.

**3. Web application security.** The OWASP Top 10 (most recently formalized in the OWASP Top 10:2021,
with a 2025 update in progress) is the standard reference for web vulnerability classes: broken access
control, cryptographic failures, injection, insecure design, security misconfiguration, vulnerable and
outdated components, identification and authentication failures, software and data integrity failures,
security logging and monitoring failures, and server-side request forgery (SSRF). Learning to find and
exploit these manually — before relying on automated scanners — is what separates a tester who can
explain business impact from one who just runs a tool.

**4. Scripting and automation.** Python and Bash are the two languages that show up constantly in
offensive tooling — Python for writing custom exploits, parsing output, and interacting with APIs; Bash
for chaining Linux tools and automating recon workflows. PowerShell becomes essential once work moves
into Windows and Active Directory environments, since it is the native automation language of the
platform being attacked.

**5. Exploitation frameworks.** Metasploit is the standard open-source framework for developing,
testing, and executing exploit code against a target, and it is usually the first framework a learner
uses to understand the exploit → payload → session workflow. Cobalt Strike is the commercial standard
for adversary emulation and red team operations, used to simulate advanced, multi-stage attacks
(command-and-control, lateral movement, persistence) in authorized engagements. Understanding both the
manual technique underneath an exploit and how these frameworks automate it is important — framework
fluency without underlying understanding tends to break down against any target that deviates from the
lab environment.

**6. Active Directory attacks.** Because most mid-size and large organizations run Windows AD for
identity and access management, AD attack paths are a core specialization for internal penetration
testing and red teaming. Key tools in this space:
- **BloodHound** — maps AD trust relationships, group memberships, and permission chains as a graph, so
  a tester can visually identify attack paths (e.g., a low-privilege user with reachable rights to
  Domain Admin) instead of enumerating them all by hand.
- **Mimikatz** — a credential-extraction and manipulation tool that demonstrates how Windows stores and
  can leak credentials in memory (e.g., via LSASS), commonly used to illustrate pass-the-hash and
  pass-the-ticket techniques in a lab context.
- **Impacket** — a collection of Python classes for working directly with network protocols used by
  Windows (SMB, MSRPC, Kerberos), underpinning many of the manual and semi-automated AD attack
  techniques such as remote command execution and ticket manipulation.

**7. CTF practice.** Capture-the-flag platforms — TryHackMe itself, Hack The Box, and similar — are
where the preceding layers get exercised together against a realistic, gamified target instead of in
isolation. CTF practice is where recon, exploitation, privilege escalation, and lateral movement stop
being separate topics and start being one continuous workflow.

### Anchoring Technique Knowledge: MITRE ATT&CK

Rather than learning exploitation techniques as disconnected tricks, offensive practitioners map what
they do to the **MITRE ATT&CK** framework — a publicly maintained, community-sourced knowledge base of
adversary tactics and techniques based on real-world observations. ATT&CK organizes attacker behavior
into tactics (the "why," e.g., Initial Access, Privilege Escalation, Lateral Movement, Exfiltration) and
techniques (the "how" within each tactic). Using ATT&CK as a reference gives a learner a shared
vocabulary with blue teams and threat intelligence analysts, and it is the same framework red team
engagements are frequently scoped and reported against.

### The Certification Ladder

A common, roughly ordered progression for validating offensive skills:

- **eJPT (eLearnSecurity Junior Penetration Tester, INE)** — an entry-level, hands-on certification
  covering penetration testing and information security essentials; a reasonable first practical
  credential after building fundamentals.
- **PJPT / PNPT (Practical Network Penetration Tester, TCM Security)** — practical, exam-based
  certifications that assess the ability to perform external and internal network penetration tests
  end-to-end, including a written report — closer to simulating a real paid engagement.
- **OSCP (Offensive Security Certified Professional, OffSec)** — widely regarded as an industry
  benchmark for hands-on penetration testing ability; the exam requires compromising multiple machines
  in a live lab within a fixed time window and documenting the process, testing both technical skill
  and the ability to work under exam pressure.

Each rung is meant to be attempted after the fundamentals below it are solid — the exams are practical
and do not reward memorization without lab time.

## Why It Matters for Security

Offensive security exists to answer a question defenders cannot fully answer on their own: what can an
attacker actually reach, and how? Penetration testers and red teamers find the gap between how a system
is *supposed* to be configured and how it actually behaves under adversarial pressure — a gap that
automated vulnerability scanners routinely miss because they cannot chain low-severity findings into a
high-severity attack path the way a human (or an AD attack path in BloodHound) can. This is also why
offensive and defensive security are two views of the same problem: MITRE ATT&CK, the shared technique
taxonomy offensive practitioners use, is the same taxonomy blue teams use to build detections — see
[Become a Defender](../become-a-defender/README.md) for the mirror image of this skill tree.

## Common Mistakes

- **Jumping straight to exploitation frameworks without networking or OS fundamentals.** Metasploit and
  Cobalt Strike automate technique execution, not understanding — without the fundamentals, a failed
  exploit against a non-lab target is a dead end rather than a debugging opportunity.
- **Tool-only learning.** Running BloodHound or Mimikatz against a lab without understanding what
  Kerberos, NTLM, or LSASS actually do produces someone who can follow a checklist but cannot adapt when
  the target environment differs even slightly from the walkthrough.
- **Skipping scripting.** Every real engagement eventually hits a situation with no existing tool for it;
  Python and Bash fluency is what turns "I don't have a tool for this" into "I wrote a ten-line script for
  this."
- **Treating CTFs as the finish line rather than practice.** CTF machines are intentionally solvable and
  often unrealistic in scope; real engagements involve ambiguity, client constraints, and reporting that
  CTFs do not train for.
- **Ignoring reporting and communication skills.** A vulnerability that cannot be clearly explained to a
  non-technical stakeholder, with accurate business impact, does not get fixed — regardless of how
  technically impressive the exploit chain was.

## Related TryHackMe Rooms in This Series

- [Become a Defender](../become-a-defender/README.md)
- [Offensive Security Intro](../../easy/offensive-security-intro/README.md)
- [Defensive Security Intro](../../easy/defensive-security-intro/README.md)
- [Careers in Cyber](../careers-in-cyber/README.md)
- [Cryptography Concepts](../cryptography-concepts/README.md)

## References

- OWASP Top 10:2021, *Introduction*: https://owasp.org/Top10/2021/A00_2021_Introduction/
- OWASP Top Ten Project: https://owasp.org/www-project-top-ten/
- MITRE ATT&CK: https://attack.mitre.org/
- MITRE D3FEND: https://d3fend.mitre.org/
- Metasploit Framework documentation: https://docs.metasploit.com/
- Impacket (SecureAuth / Fortra): https://github.com/fortra/impacket
- BloodHound documentation: https://bloodhound.readthedocs.io/
- OffSec, *OSCP Certification*: https://www.offsec.com/courses/pen-200/
- TCM Security, *PNPT Certification*: https://certifications.tcm-sec.com/pnpt/
- INE, *eJPT Certification*: https://ine.com/certifications/ejpt-certification
