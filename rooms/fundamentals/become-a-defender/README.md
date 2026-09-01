# Become a Defender

**Platform:** TryHackMe · **Type:** Concept Guide (no personal run captured — see note below)
**Primary domain:** Security Fundamentals

> **Note:** This is a concept guide covering the material this TryHackMe room teaches, written from
> public documentation and reference material. It is not a personal walkthrough — no session evidence
> (screenshots, command output, room-specific answers) is included.

## Overview

"Become a Defender" is TryHackMe's orientation room for the blue team / defensive security learning
path: it lays out the skills needed to detect, investigate, and respond to attacks rather than carry
them out. Defensive security is often less visible than offensive security — there is no single
dramatic exploit moment — but it is a deeper discipline in practice, requiring familiarity with
networking, log data at scale, detection engineering, and formal incident response process. This guide
lays that skill tree out concretely, with real tool families and the certification path that validates
each stage.

## Core Concepts

### The Defensive Skill Tree

**1. Networking fundamentals.** Just as with offensive security, everything downstream depends on
understanding TCP/IP, DNS, routing, and common protocols — a defender cannot recognize anomalous
traffic without first knowing what normal traffic looks like.

**2. Log analysis.** Every meaningful security event leaves a trail across firewall logs, authentication
logs, DNS logs, web server logs, and endpoint logs. Learning to read, correlate, and query these
formats — and to recognize what "normal" log volume and content looks like for a given environment — is
the foundation that every higher-level detection tool sits on top of.

**3. SIEM usage.** A Security Information and Event Management (SIEM) platform centralizes log
ingestion, correlation, and alerting across an environment. The three platforms a defender is most
likely to encounter in the field:
- **Splunk** — a widely deployed commercial platform built around its Search Processing Language (SPL)
  for querying and correlating large volumes of log data.
- **Elastic Stack (Elasticsearch, Logstash, Kibana)** — an open-source alternative commonly deployed as
  the "ELK stack," used both for SIEM-style detection and for general log analytics.
- **Microsoft Sentinel** — a cloud-native SIEM and SOAR (security orchestration, automation, and
  response) platform built on Azure, notable for deep integration with Microsoft 365 and Entra ID
  telemetry.

**4. Endpoint detection (EDR concepts).** Endpoint Detection and Response tools monitor process
creation, file changes, network connections, and registry activity on individual hosts, then surface
suspicious behavior chains rather than relying on static signatures alone. **Sysmon** (System Monitor),
a free Windows Sysinternals tool, is the most common way to generate the detailed endpoint telemetry
(process creation with full command lines, network connections, file creation time changes) that feeds
both commercial EDR products and SIEM detection rules — understanding what Sysmon logs, and why, is
foundational before touching any commercial EDR console.

**5. Threat hunting.** Where detection is reactive (an alert fires), threat hunting is proactive:
formulating a hypothesis about how an adversary might already be present — often informed by MITRE
ATT&CK technique IDs — and searching log and endpoint data to prove or disprove it, rather than waiting
for a rule to trigger.

**6. Incident response.** NIST Special Publication 800-61 is the standard reference for incident
response process. Its long-established four-phase lifecycle — still the most commonly taught model,
even after NIST's 2025 Revision 3 realigned the publication to the six functions of the NIST
Cybersecurity Framework 2.0 — is:
1. **Preparation** — building the people, process, and tooling (playbooks, contact lists, forensic
   readiness) needed before an incident happens.
2. **Detection and Analysis** — recognizing that an incident is occurring and scoping what it affects.
3. **Containment, Eradication, and Recovery** — stopping the spread, removing the attacker's foothold,
   and restoring affected systems to normal operation.
4. **Post-Incident Activity** — the retrospective: lessons learned, updating playbooks and detections,
   and formally closing the incident.

**7. Digital forensics basics.** Forensics is what turns a compromised host into evidence: reconstructing
what happened, when, and how, in a way that can support both the technical remediation and (if needed)
legal or HR processes. Key tool families:
- **Velociraptor** — an open-source endpoint visibility and collection platform used for large-scale,
  remote forensic triage and hunting across many hosts at once.
- **KAPE (Kroll Artifact Parser and Extractor)** — a fast forensic artifact collection and processing
  tool used to pull and parse the specific files and registry keys that matter most during an
  investigation, without imaging an entire disk.
- **YARA** — a pattern-matching tool for writing rules that identify malware families or specific
  malicious files based on binary and textual patterns, used both in forensic analysis and in live
  detection pipelines.

Underneath all of this, **Wireshark** (deep-dive packet inspection) and **Zeek** (network security
monitoring that produces structured, high-level logs of network activity rather than raw packets) are
the two tool families most associated with network-layer monitoring and investigation.

### Anchoring Technique Knowledge: MITRE ATT&CK and D3FEND

Defenders use **MITRE ATT&CK** the same way offensive practitioners do — as a shared vocabulary for
adversary tactics and techniques — but from the opposite direction: mapping detections and hunts to
specific technique IDs so coverage gaps become visible. **MITRE D3FEND**, a complementary knowledge
graph funded by the NSA and developed by MITRE, catalogs defensive countermeasures and maps them
directly to the ATT&CK techniques they counter, organized around five defensive tactics: Harden,
Detect, Isolate, Deceive, and Evict. Where ATT&CK answers "what might an attacker do," D3FEND answers
"what specific control addresses that."

### The Certification Ladder

A common, roughly ordered progression for validating defensive skills:

- **Security+ (CompTIA)** — the standard entry-level, vendor-neutral certification covering core
  security concepts, network security, and risk management; a typical prerequisite before specializing.
- **GCIH (GIAC Certified Incident Handler)** — validates incident handling skills, tied to SANS course
  SEC504; focused on detecting, responding to, and containing active attacks.
- **GCIA (GIAC Certified Intrusion Analyst)** — validates network traffic analysis and intrusion
  detection skills, tied to SANS course SEC503.
- **GCFA (GIAC Certified Forensic Analyst)** and **GCFE (GIAC Certified Forensic Examiner)** — advanced
  forensics and threat-hunting (GCFA, SANS FOR508) and Windows forensics fundamentals (GCFE, SANS
  FOR500), respectively, for practitioners specializing in DFIR (digital forensics and incident
  response).

## Why It Matters for Security

Defensive security is what makes offensive findings actionable at scale: an organization that
understands its own attack surface (from penetration testing) but has no detection or response
capability will still get breached, just more slowly noticed. Incident response process — NIST SP
800-61's phases in particular — exists because ad hoc, panic-driven response to a live incident reliably
makes outcomes worse: evidence gets destroyed, containment happens too late or too early, and lessons
learned get lost. Framing detection engineering and threat hunting around MITRE ATT&CK (and validating
coverage against D3FEND) is what turns "we have a SIEM" into "we have measurable coverage against known
adversary behavior" — the difference between having tools and having a defensible security program.

## Common Pitfalls / Misconfigurations

- **Alert fatigue from poorly tuned SIEM rules.** A SIEM producing thousands of low-fidelity alerts
  trains analysts to ignore alerts generally, which is how genuine incidents get missed.
- **No baseline for "normal."** Without an established baseline of typical network and endpoint
  behavior, anomaly-based detection is guesswork, and threat hunting has nothing to hypothesize against.
- **Sysmon deployed with default configuration.** The default Sysmon config logs relatively little of
  value; effective deployments use a curated configuration (such as the widely used SwiftOnSecurity
  base) tuned to the environment's actual telemetry needs.
- **Skipping the Preparation phase of IR.** Teams that only think about incident response once an
  incident is underway lose critical time to building playbooks and contact lists that should have
  existed beforehand.
- **Treating containment and eradication as optional once detection happens.** Detecting an intrusion
  without a documented containment and recovery plan often results in an attacker simply re-entering
  through the same foothold.
- **No post-incident review.** Skipping the Post-Incident Activity phase means the same gap that allowed
  the incident tends to remain open for the next attacker.

## Related TryHackMe Rooms in This Series

- [Become a Hacker](../become-a-hacker/README.md)
- [Defensive Security Intro](../../easy/defensive-security-intro/README.md)
- [Offensive Security Intro](../../easy/offensive-security-intro/README.md)
- [Careers in Cyber](../careers-in-cyber/README.md)
- [Cryptography Concepts](../cryptography-concepts/README.md)

## References

- NIST SP 800-61 Rev. 2, *Computer Security Incident Handling Guide*: https://csrc.nist.gov/pubs/sp/800/61/r2/final
- NIST SP 800-61 Rev. 3, *Incident Response Recommendations and Considerations for Cybersecurity Risk Management*: https://csrc.nist.gov/pubs/sp/800/61/r3/final
- NIST Cybersecurity Framework 2.0: https://www.nist.gov/cyberframework
- MITRE ATT&CK: https://attack.mitre.org/
- MITRE D3FEND: https://d3fend.mitre.org/
- Sysmon (Sysinternals): https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon
- Zeek Network Security Monitor: https://zeek.org/
- Velociraptor documentation: https://docs.velociraptor.app/
- KAPE (Kroll Artifact Parser and Extractor): https://www.kroll.com/en/services/cyber-risk/incident-response-litigation-support/kroll-artifact-parser-extractor-kape
- YARA documentation: https://yara.readthedocs.io/
- GIAC Certifications: https://www.giac.org/certifications/
- CompTIA Security+: https://www.comptia.org/certifications/security
