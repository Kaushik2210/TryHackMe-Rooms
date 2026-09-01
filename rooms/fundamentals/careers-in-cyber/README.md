# Careers in Cyber

**Platform:** TryHackMe · **Type:** Concept Guide (no personal run captured — see note below)
**Primary domain:** Security Fundamentals

> **Note:** This is a concept guide covering the material this TryHackMe room teaches, written from
> public documentation and reference material. It is not a personal walkthrough — no session evidence
> (screenshots, command output, room-specific answers) is included.

## Overview

Cybersecurity is not one job — it is a collection of distinct disciplines (defensive monitoring,
offensive testing, governance, incident response, intelligence) that share a common knowledge base but
diverge sharply in day-to-day work, required skills, and entry paths. NIST's Workforce Framework for
Cybersecurity (NICE Framework, published as NIST SP 800-181 Revision 1) is the standard taxonomy used
across US government, industry, and academia to describe this work: it organizes cybersecurity labor
into seven high-level categories — Analyze, Collect & Operate, Investigate, Operate & Maintain, Oversee
& Govern, Protect & Defend, and Securely Provision — each broken into specific work roles with defined
tasks, knowledge, and skill statements. This guide walks through five roles that map cleanly onto that
framework and that most newcomers gravitate toward first.

## Core Concepts

### SOC Analyst (Tier 1–3)

Security Operations Center analysts sit under NICE's **Protect & Defend** category (work role: Cyber
Defense Analyst) and form the backbone of most enterprise security teams. The role is tiered by
experience and scope:

- **Tier 1 (Triage):** Monitors SIEM dashboards (Splunk, Microsoft Sentinel, QRadar), performs initial
  alert triage against known indicators of compromise, and escalates true positives while filtering
  false positives. Entry-level; typically the first paid role in a SOC career.
- **Tier 2 (Investigation):** Takes escalated alerts, correlates events across multiple log sources,
  performs root-cause analysis, and executes initial containment (isolating a host, disabling an
  account). Requires deeper knowledge of networking, endpoint forensics basics, and attacker TTPs
  (MITRE ATT&CK).
- **Tier 3 (Threat Hunting / Engineering):** Proactively hunts for threats that evade existing
  detections, writes new detection content and correlation rules, and often bridges into detection
  engineering or incident response.

**Tools:** SIEM platforms, EDR consoles (CrowdStrike, Microsoft Defender), packet analysis (Wireshark),
ticketing systems.
**Entry requirements:** Associate's degree or equivalent, or a bootcamp/self-study path; strong
networking fundamentals (TCP/IP, DNS, HTTP); no prior professional experience strictly required for
Tier 1.
**Certifications:** CompTIA Security+ (baseline, often a hiring filter), CompTIA CySA+ (analyst-focused),
GIAC Certified Incident Handler (GCIH) for Tier 2/3.
**Progression:** Tier 1 → Tier 2 → Tier 3 → SOC Lead / Detection Engineer / Incident Responder, usually
over 3–6 years.

### Penetration Tester

Sits under NICE's **Securely Provision** category (work role: Vulnerability Assessment Analyst /
Exploitation Analyst overlap). Penetration testers simulate real attacks against networks,
applications, or cloud environments under a signed scope of engagement, then document findings with
reproduction steps and remediation guidance for the client.

**Day-to-day tasks:** Scoping engagements, reconnaissance and enumeration, exploiting vulnerabilities to
demonstrate real business impact (not just scanning), privilege escalation, and — critically — writing
a clear, client-facing report, which consumes as much time as the technical testing itself.
**Tools:** Nmap, Burp Suite, Metasploit, Cobalt Strike (red team), BloodHound (Active Directory attack
paths), custom scripting (Python/PowerShell).
**Entry requirements:** Strong fundamentals in networking, Linux/Windows administration, and at least
one scripting language; a home lab (TryHackMe, HackTheBox) and CTF experience are commonly used as
practical proof of skill in place of formal experience.
**Certifications:** CompTIA PenTest+ (entry), Offensive Security Certified Professional (OSCP) — widely
regarded as the industry benchmark for hands-on exploitation skill, GIAC Penetration Tester (GPEN),
and eLearnSecurity/INE eJPT as an earlier stepping stone.
**Progression:** Junior pentester → pentester → senior/lead pentester → red team operator or
principal consultant; many eventually move into security research or founding a boutique testing firm.

### GRC (Governance, Risk & Compliance) Analyst

Sits under NICE's **Oversee & Govern** category. GRC analysts translate security requirements into
policy, measure organizational risk, and ensure the business meets regulatory and contractual
obligations (SOC 2, ISO 27001, PCI DSS, HIPAA, GDPR, depending on sector).

**Day-to-day tasks:** Running risk assessments (using frameworks like NIST SP 800-30), maintaining
policy and control documentation, coordinating internal and external audits, tracking remediation of
audit findings, and translating technical vulnerabilities into business risk language for executives
and boards.
**Tools:** GRC platforms (ServiceNow GRC, Vanta, Drata), spreadsheets/risk registers, control frameworks
(NIST CSF, ISO 27001 Annex A, CIS Controls).
**Entry requirements:** Often a business, IT, or legal/compliance background rather than a purely
technical one; strong writing and stakeholder-communication skills matter as much as technical depth.
**Certifications:** ISACA Certified Information Systems Auditor (CISA), ISACA Certified in Risk and
Information Systems Control (CRISC), ISC2 Certified Information Systems Security Professional (CISSP)
at the senior level, ISO 27001 Lead Auditor.
**Progression:** GRC analyst → GRC/compliance manager → Chief Information Security Officer (CISO) or
Chief Risk Officer track — GRC is one of the more common non-technical paths into executive security
leadership.

### Incident Responder

Sits under NICE's **Protect & Defend** category (work role: Incident Responder), one step beyond the
SOC. Incident responders take confirmed, high-severity incidents and manage them through containment,
eradication, and recovery, typically following a structured lifecycle such as NIST SP 800-61's
Preparation → Detection & Analysis → Containment/Eradication/Recovery → Post-Incident Activity model.

**Day-to-day tasks:** Leading live incident response calls, performing memory and disk forensics to
determine scope and root cause, coordinating with legal/PR/executive stakeholders during a breach,
writing post-incident reports and lessons-learned documentation, and building playbooks that speed up
future response.
**Tools:** EDR platforms, forensic suites (Volatility for memory analysis, Autopsy/EnCase for disk),
SOAR platforms for playbook automation, log aggregation (Splunk, Elastic).
**Entry requirements:** Usually 2+ years in a SOC or systems administration role first; strong
Windows/Linux internals knowledge and familiarity with attacker TTPs (MITRE ATT&CK) are expected.
**Certifications:** GIAC Certified Incident Handler (GCIH), GIAC Certified Forensic Analyst (GCFA),
EC-Council Certified Incident Handler (ECIH).
**Progression:** SOC Tier 2/3 → Incident Responder → Senior IR / DFIR (Digital Forensics & Incident
Response) lead → IR manager or CSIRT team lead.

### Threat Intelligence Analyst

Sits under NICE's **Analyze** category (work role: All-Source/Cyber Threat Analyst). This role studies
threat actors, campaigns, and infrastructure to produce actionable intelligence that informs defensive
priorities — closer to an analytical/research discipline than a hands-on-keyboard defensive one.

**Day-to-day tasks:** Tracking threat actor groups and their tactics, techniques, and procedures (TTPs)
against the MITRE ATT&CK framework, producing intelligence reports for executives and SOC teams,
managing indicator feeds (IOCs) for ingestion into SIEM/EDR, and briefing leadership on emerging risks
relevant to the organization's sector.
**Tools:** Threat intel platforms (MISP, Recorded Future, Anomali), OSINT tooling, ATT&CK Navigator,
dark web monitoring services.
**Entry requirements:** Strong analytical writing skills, research discipline, and a solid grounding in
how attacks actually unfold technically; often a lateral move from a SOC or IR role rather than a pure
entry point.
**Certifications:** GIAC Cyber Threat Intelligence (GCTI), Certified Threat Intelligence Analyst (CTIA),
CompTIA Security+ as a baseline.
**Progression:** SOC/IR analyst → Threat Intelligence Analyst → Senior Threat Intel / Threat Hunting
lead → Threat Intelligence program manager.

## Why It Matters for Security

Understanding these role distinctions matters practically, not just for career planning: it determines
who owns what during an incident, what a job posting is actually asking for, and which skills to build
first. A candidate who studies exploitation techniques when applying for GRC roles — or who studies
audit frameworks when applying for pentesting roles — will consistently underperform in interviews
relative to their raw ability, because the two disciplines test almost entirely different competencies.
The NICE Framework exists precisely to reduce this ambiguity: it gives employers a consistent way to
write job descriptions and gives job-seekers a consistent way to map their existing skills (or a
learning plan like this room) onto real, hireable roles rather than a vague "cybersecurity" label.

## Common Mistakes

- **Chasing certifications before fundamentals.** A stack of certificates without solid networking,
  Linux, and scripting fundamentals underneath is a common reason technically-oriented interviews go
  poorly — certifications should validate skill built through practice, not substitute for it.
- **Treating "cybersecurity" as one job when applying.** Applying broadly to "cybersecurity analyst"
  postings without reading whether the role is SOC, GRC, or pentest-flavored wastes both the
  candidate's and recruiter's time and produces mismatched interviews.
- **Skipping the SOC/helpdesk stepping stone.** Many roles (pentesting, incident response, threat
  intel) expect prior operational experience; trying to enter directly into a senior specialist role
  with no foundational IT or SOC time is a frequent source of rejected applications.
- **Underestimating the writing/communication component.** GRC, threat intelligence, incident response,
  and even pentesting all live or die on report quality and stakeholder communication — purely technical
  skill without the ability to write a clear findings report or risk brief caps career progression.
- **Ignoring the NICE Framework (or an equivalent skills map) when self-studying.** Without a reference
  taxonomy, it's easy to build a scattershot skill set that doesn't map to any real, hireable work role.

## Related TryHackMe Rooms in This Series

- [Become a Hacker](../become-a-hacker/README.md)
- [Become a Defender](../become-a-defender/README.md)
- [Defensive Security Intro](../../easy/defensive-security-intro/README.md)
- [Offensive Security Intro](../../easy/offensive-security-intro/README.md)

## References

- NIST SP 800-181 Rev. 1, *Workforce Framework for Cybersecurity (NICE Framework)*: https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-181r1.pdf
- NICCS (CISA), *NICE Workforce Framework for Cybersecurity*: https://niccs.cisa.gov/tools/nice-framework
- NIST, *NICE Framework Resource Center*: https://www.nist.gov/itl/applied-cybersecurity/nice/nice-framework-resource-center
- NIST SP 800-61 Rev. 2, *Computer Security Incident Handling Guide*: https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf
- NIST SP 800-30 Rev. 1, *Guide for Conducting Risk Assessments*: https://csrc.nist.gov/pubs/sp/800/30/r1/final
- CompTIA, *Security+ Certification*: https://www.comptia.org/certifications/security
- CompTIA, *PenTest+ Certification*: https://www.comptia.org/certifications/pentest
- Offensive Security, *OSCP Certification*: https://www.offsec.com/courses/pen-200/
- GIAC, *GCIH – Certified Incident Handler*: https://www.giac.org/certifications/certified-incident-handler-gcih/
- GIAC, *GCFA – Certified Forensic Analyst*: https://www.giac.org/certifications/certified-forensic-analyst-gcfa/
- GIAC, *GCTI – Cyber Threat Intelligence*: https://www.giac.org/certifications/cyber-threat-intelligence-gcti/
- ISC2, *CISSP – Certified Information Systems Security Professional*: https://www.isc2.org/certifications/cissp
- ISACA, *CISA – Certified Information Systems Auditor*: https://www.isaca.org/credentialing/cisa
- ISACA, *CRISC – Certified in Risk and Information Systems Control*: https://www.isaca.org/credentialing/crisc
- MITRE, *ATT&CK Framework*: https://attack.mitre.org/
