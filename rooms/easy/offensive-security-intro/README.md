# Offensive Security Intro

**Platform:** TryHackMe · **Type:** Concept Guide (no personal run captured — see note below)
**Primary domain:** Security Fundamentals

> **Note:** This is a concept guide covering the material this TryHackMe room teaches, written from
> public documentation and reference material. It is not a personal walkthrough — no session evidence
> (screenshots, command output, room-specific answers) is included.

## Overview

Offensive security ("red team" work) is the practice of legally and deliberately simulating attacker
behavior — reconnaissance, exploitation, and post-exploitation — to find weaknesses before real
adversaries do. It ranges from single-vulnerability penetration tests to full adversary-emulation
engagements that mimic a specific threat actor's tradecraft. This guide covers the two dominant models
for describing an attack (the Cyber Kill Chain and MITRE ATT&CK), the phases a formal penetration test
follows, and the tool families a red teamer relies on at each stage.

## Core Concepts

### The Cyber Kill Chain

Lockheed Martin adapted the term "kill chain" from military targeting doctrine and published it in
2011 as a model for describing the stages an intrusion moves through, on the premise that a defender
only needs to break one link to stop the whole chain while an attacker must succeed at every stage.
The seven stages are:

1. **Reconnaissance** — Gathering information about the target: passive collection from public
   sources (OSINT, social media, DNS records, job postings) and active probing (port scanning, service
   fingerprinting).
2. **Weaponization** — Coupling an exploit with a payload into a deliverable artifact, such as a
   malicious document with an embedded macro or a trojanized installer.
3. **Delivery** — Transmitting the weaponized artifact to the target, commonly via phishing email,
   a watering-hole website, or a USB drop.
4. **Exploitation** — Triggering the vulnerability or logic flaw so the payload executes on the target
   system, gaining the attacker initial code execution.
5. **Installation** — Establishing persistence, such as installing a backdoor or scheduled task so the
   attacker retains access across reboots.
6. **Command and Control (C2)** — Opening an outbound channel from the compromised host to
   attacker-controlled infrastructure so the attacker can issue further commands.
7. **Actions on Objectives** — Carrying out the actual goal of the intrusion: data exfiltration,
   lateral movement toward a higher-value target, destruction, or extortion via ransomware.

The kill chain's main value is conceptual: it gives defenders a shared vocabulary for "how far did this
intrusion get" and highlights that early-stage disruption (blocking reconnaissance or delivery) is
cheaper than late-stage cleanup after actions on objectives. Its main limitation, widely noted in the
industry, is that it was built around a fairly linear, malware-delivery-centric model of intrusion and
maps less cleanly onto cloud-native, credential-based, or insider-driven attacks — which is part of why
MITRE ATT&CK has become the more commonly used working reference for red and blue teams alike.

### MITRE ATT&CK Framework

MITRE ATT&CK (Adversarial Tactics, Techniques, and Common Knowledge) is a publicly maintained
knowledge base of real-world adversary behavior, structured as a matrix rather than a linear chain.
Its Enterprise matrix — the most commonly referenced one — currently documents 14 tactics covering the
full attack lifecycle (from Reconnaissance and Initial Access through Execution, Persistence,
Privilege Escalation, Defense Evasion, Credential Access, Discovery, Lateral Movement, Collection,
Command and Control, Exfiltration, and Impact), populated with over 200 techniques and hundreds of
sub-techniques.

- **Tactics** are the columns of the matrix and represent the attacker's *why* — the tactical goal at
  a given stage (e.g., Credential Access).
- **Techniques and sub-techniques** are the cells beneath each tactic and represent the *how* — a
  specific method used to achieve that goal (e.g., OS Credential Dumping, with sub-techniques for
  LSASS memory, SAM database, or NTDS extraction).
- Each technique entry documents known procedure examples (which real threat groups have used it),
  detection guidance, and mitigations, which is what makes ATT&CK actionable for both sides: red teams
  use it to plan realistic emulation plans and select techniques matching a specific adversary group's
  known behavior, while blue teams use it to score detection coverage — which techniques does the SOC
  actually have alerting for — and to prioritize detection engineering against gaps.

Separate matrices exist for Mobile (Android/iOS) and ICS (industrial control systems/OT environments),
reflecting that adversary tradecraft differs meaningfully by platform.

### Penetration Test Phases

Formal engagements follow a structured methodology, most commonly referenced against NIST SP 800-115
(*Technical Guide to Information Security Testing and Assessment*) or the Penetration Testing
Execution Standard (PTES), both of which describe broadly the same lifecycle:

1. **Reconnaissance (Information Gathering)** — Passive and active collection to build a picture of
   the target's attack surface: domains, IP ranges, employee names, exposed services, and technology
   stack.
2. **Scanning and Enumeration** — Actively probing discovered assets to identify live hosts, open
   ports, running services and versions, and misconfigurations — turning a broad attack surface into a
   prioritized list of specific things to test.
3. **Exploitation** — Attempting to leverage identified vulnerabilities to gain unauthorized access,
   whether that's a web application flaw, an unpatched service, or a weak credential.
4. **Post-Exploitation** — Once initial access is gained, escalating privileges, moving laterally to
   other systems, and demonstrating impact (such as reaching data that maps to the engagement's defined
   objectives) without exceeding the agreed rules of engagement.
5. **Reporting** — Documenting findings with reproducible evidence, business risk context, and
   remediation guidance. NIST SP 800-115 and PTES both treat reporting as a core phase rather than an
   afterthought, since a technically successful test that produces an unusable report delivers no
   security value to the client.

Every phase operates within a defined scope and rules of engagement agreed with the client in advance,
and legitimate engagements require explicit written authorization — this is the line that separates
penetration testing from unauthorized intrusion under laws such as the U.S. Computer Fraud and Abuse
Act.

### Red Team Tool Families

Offensive tooling maps closely to the phases above:

- **Nmap** — The standard tool for network discovery and port/service scanning, used heavily during
  the scanning and enumeration phase to fingerprint live hosts, open ports, and service versions before
  narrowing in on exploitation targets.
- **Burp Suite** — A web application testing proxy that intercepts, inspects, and modifies HTTP(S)
  traffic between browser and server, used to identify and exploit web-layer vulnerabilities (injection
  flaws, authentication bypasses, business logic issues) that automated scanners alone tend to miss.
- **Metasploit** — An exploitation framework that packages known vulnerabilities as reusable modules
  alongside payload generation and post-exploitation tooling, commonly used to validate that a
  discovered vulnerability is actually exploitable rather than merely present.
- **Cobalt Strike** — A commercial adversary-simulation platform built around a "beacon" C2 agent,
  used primarily in red team engagements that emulate a specific threat actor's post-exploitation and
  command-and-control tradecraft rather than a single-vulnerability pentest; its prevalence in real
  intrusions (both legitimate red teams and cracked/leaked copies used by criminal groups) also makes
  it a frequent subject of blue team detection engineering.

Each of these tool categories corresponds to techniques cataloged in MITRE ATT&CK, which is part of why
mature red teams plan engagements by selecting ATT&CK techniques to emulate rather than starting from
"which tool should I run."

## Why It Matters for Security

Offensive security exists to answer a question defenders cannot answer from their own side: not "do we
have controls," but "do our controls actually stop a competent attacker." A vulnerability scan reports
what's theoretically exploitable; a penetration test or red team engagement proves what's practically
exploitable and, in a mature engagement, whether the SOC actually detected and responded to it. This is
why offensive and defensive security are increasingly run as a joint exercise — "purple teaming" —
where red team activity is mapped to specific MITRE ATT&CK techniques so the blue team can validate
detection coverage against them in near-real time, closing the loop between finding a gap and proving
it's fixed.

## Common Pitfalls / Misconfigurations

- **Testing without a defined scope or authorization.** Running offensive tooling against systems
  without explicit written permission is illegal regardless of intent, and scope creep during an
  authorized engagement is one of the most common sources of client disputes.
- **Treating a vulnerability scan as a penetration test.** Automated scanning identifies candidate
  weaknesses; it does not validate exploitability, chain vulnerabilities together, or demonstrate
  business impact the way manual testing does.
- **Skipping reconnaissance depth.** Jumping straight to exploitation tools without adequate
  enumeration means missing lower-effort attack paths (default credentials, exposed admin panels,
  misconfigured cloud storage) in favor of harder ones.
- **Reports with no reproducible evidence or remediation guidance.** A finding that can't be
  reproduced or acted on by the client's engineering team fails to deliver the actual value of the
  engagement, regardless of how technically impressive the exploitation was.
- **Over-reliance on a single framework.** Mapping only to the Cyber Kill Chain misses lateral
  movement, credential-based, and cloud-native attack patterns that MITRE ATT&CK's broader tactic set
  captures more completely — most mature programs use ATT&CK as the primary reference and the kill
  chain as a simpler explanatory model for non-technical stakeholders.

## Related TryHackMe Rooms in This Series

- [Defensive Security Intro](../defensive-security-intro/README.md)
- [The CIA Triad](../the-cia-triad/README.md)
- [Cryptography Concepts](../../fundamentals/cryptography-concepts/README.md)

## References

- Lockheed Martin, *Cyber Kill Chain*: https://www.lockheedmartin.com/en-us/capabilities/cyber/cyber-kill-chain.html
- MITRE ATT&CK, *Enterprise Matrix*: https://attack.mitre.org/matrices/enterprise/
- MITRE ATT&CK, *Design and Philosophy*: https://attack.mitre.org/resources/attack-design-and-philosophy/
- NIST SP 800-115, *Technical Guide to Information Security Testing and Assessment*: https://csrc.nist.gov/pubs/sp/800/115/final
- The Penetration Testing Execution Standard (PTES): http://www.pentest-standard.org/index.php/Main_Page
- Nmap, official project documentation: https://nmap.org/
- Metasploit, official project documentation: https://www.metasploit.com/
