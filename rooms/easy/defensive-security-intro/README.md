# Defensive Security Intro

**Platform:** TryHackMe · **Type:** Concept Guide (no personal run captured — see note below)
**Primary domain:** Security Fundamentals

> **Note:** This is a concept guide covering the material this TryHackMe room teaches, written from
> public documentation and reference material. It is not a personal walkthrough — no session evidence
> (screenshots, command output, room-specific answers) is included.

## Overview

Defensive security (also called "blue team" work) is the set of practices, roles, and tooling
organizations use to prevent, detect, and respond to cyber threats before, during, and after they
occur. Where offensive security simulates attacks, defensive security builds and operates the
people, process, and technology that catch and contain real ones. This guide covers how a Security
Operations Center (SOC) is structured, the NIST Cybersecurity Framework used to organize a defensive
program, and the core detection and tooling concepts a working blue teamer relies on day to day.

## Core Concepts

### SOC Tiering and Escalation Flow

A Security Operations Center is typically staffed in tiers, each with a distinct scope of
responsibility. This structure exists so that high-volume, low-complexity alerts are triaged quickly
without consuming the time of the most experienced (and most expensive) analysts.

- **Tier 1 — Triage Analyst.** Monitors the SIEM/alert queue, performs initial triage against
  playbooks, and classifies alerts as false positive, benign true positive, or true positive requiring
  escalation. Tier 1 analysts typically handle high alert volume and low-context decisions — is this a
  known-bad IP, does this login match a normal pattern — and escalate anything ambiguous or confirmed
  malicious to Tier 2.
- **Tier 2 — Incident Responder.** Performs deeper investigation on escalated alerts: correlating logs
  across multiple sources, scoping the blast radius of a suspected compromise, and executing the
  initial containment steps (isolating a host, disabling an account, blocking an indicator). Tier 2
  owns the bulk of active incident response and typically follows a structured process such as the
  one described in NIST SP 800-61 (preparation, detection and analysis, containment/eradication/
  recovery, post-incident activity).
- **Tier 3 — Threat Hunter / Senior Analyst.** Performs proactive threat hunting rather than
  alert-driven work, reverse engineers malware samples, builds and tunes detection content, and leads
  the technical response to major incidents. Tier 3 is also usually responsible for detection
  engineering — writing and validating the rules that generate Tier 1's alert queue in the first
  place.
- **Tier 4 (in larger organizations) — SOC Manager.** Owns process, metrics (mean time to detect/
  respond), staffing, and coordination with legal, PR, and executive stakeholders during major
  incidents.

Escalation flows upward: Tier 1 triages and hands off what it cannot resolve within its playbook
scope, Tier 2 investigates and contains, and Tier 3 hunts, engineers detections, and handles the
hardest cases. Smaller organizations often collapse tiers 2 and 3 into a single "senior analyst" role,
but the functional split — triage, respond, hunt/engineer — holds regardless of headcount.

### NIST Cybersecurity Framework (CSF) 2.0

The NIST Cybersecurity Framework, most recently updated to version 2.0 in February 2024, is the most
widely referenced model for organizing a defensive security program end to end. CSF 2.0 restructured
the original five functions into six by adding a new function, **Govern**, which sits at the center
and informs the other five:

1. **Govern (GV)** — Establishes and communicates the organization's cybersecurity risk management
   strategy, roles, responsibilities, policy, and oversight. This is new in 2.0 and reflects the
   framework's expanded scope beyond purely technical controls into governance and supply-chain risk.
2. **Identify (ID)** — Develops organizational understanding of assets, data, systems, suppliers, and
   the risks they carry, forming the inventory a defensive program is built against.
3. **Protect (PR)** — Implements safeguards to limit or contain the impact of a potential event:
   access control, awareness training, data security, and platform hardening.
4. **Detect (DE)** — Implements activities to find cybersecurity events in a timely manner —
   continuous monitoring and detection processes, which is where SOC alerting and SIEM content live.
5. **Respond (RS)** — Takes action once an incident is detected: incident management, analysis,
   mitigation, communication, and reporting.
6. **Recover (RC)** — Restores capabilities and services impaired by a cybersecurity incident and
   manages communication during recovery.

CSF 2.0 is deliberately outcome-based rather than prescriptive — it does not mandate specific tools or
configurations, which is why organizations map it against more granular control catalogs like NIST
SP 800-53 to implement each function concretely. In a SOC context, most day-to-day analyst work lives
inside Detect and Respond, while Protect and Govern are largely owned by security engineering and
GRC (governance, risk, and compliance) teams.

### Detection Engineering Basics

Detection engineering is the discipline of building, testing, and maintaining the logic that turns raw
telemetry into actionable alerts.

- **Detection-as-code.** Modern SOCs increasingly manage detection rules the way software engineers
  manage application code: rules live in version control (git), go through peer review, and are
  deployed via CI/CD pipelines rather than being edited ad hoc in a SIEM console. This makes detection
  logic testable, auditable, and rollback-able, and lets teams run regression tests against known
  attack samples before shipping a new rule.
- **Alert triage.** The process of taking a raw alert and deciding what it means: is it a true
  positive (real malicious activity), a benign true positive (real activity that is not malicious, such
  as an authorized pentest), or a false positive (the rule fired on non-malicious activity). Triage
  relies on enrichment — pulling in threat intelligence, asset criticality, and user context — to make
  that call quickly.
- **False positive tuning.** Rules that are too broad generate alert fatigue, which is one of the
  leading causes of missed true positives in real SOCs — analysts become desensitized to a queue that
  is mostly noise. Tuning narrows detection logic (adding exclusions, raising thresholds, requiring
  multiple correlated conditions) without opening gaps an attacker could walk through. This is
  typically measured and iterated on using metrics like precision (true positives / total alerts) and
  detection coverage mapped against a framework like MITRE ATT&CK.

### Blue Team Tool Families

Defensive tooling generally falls into a small number of functional categories, and most SOCs run at
least one product from each:

- **SIEM (Security Information and Event Management)** — Aggregates and correlates log data from
  across the environment and is the primary interface Tier 1/2 analysts work from. Common products
  include Splunk, Elastic (Elastic Security / ELK stack), and Microsoft Sentinel (a cloud-native SIEM
  built on Azure). SIEMs are where detection-as-code rules ultimately execute and where alert queues
  live.
- **EDR (Endpoint Detection and Response)** — Instruments individual hosts (workstations, servers) to
  capture process execution, file, registry, and network activity, and gives responders the ability to
  isolate a host or kill a process remotely. CrowdStrike Falcon and Microsoft Defender for Endpoint are
  widely deployed examples; EDR telemetry is frequently the richest data source for identifying
  post-exploitation activity such as living-off-the-land technique use.
- **IDS/IPS (Intrusion Detection/Prevention Systems)** — Inspect network traffic against signature or
  anomaly-based rules to flag or block malicious activity in transit. Snort and Suricata are the two
  dominant open-source engines and both use a similar rule syntax; Suricata additionally supports
  multi-threaded processing and native protocol logging (e.g., extracting TLS certificates or HTTP
  metadata) that make it useful for network security monitoring beyond simple alerting.
- **Log aggregation and centralization** — The foundational layer underneath a SIEM: shipping logs
  from endpoints, network devices, cloud services, and applications to a central store (via agents,
  syslog, or cloud-native log export) so they can be correlated. Without reliable, complete log
  collection, every layer above it — detection, triage, hunting — is working from an incomplete
  picture, which is why log source coverage is one of the first things a SOC audits when standing up
  or maturing its capability.

## Why It Matters for Security

Defensive security is what converts security investment into measurable risk reduction: a well-tuned
SIEM rule or a properly staffed Tier 1 queue is often what separates a contained intrusion from a
headline breach. The tiered SOC model exists because detection and response at scale is a throughput
problem as much as a technical one — organizations that skip it (routing everything to senior staff, or
running an under-tuned alert queue) either burn out their best analysts or drown in false positives
until real incidents get missed. Framing a defensive program around CSF 2.0's six functions also gives
security leaders a common vocabulary to communicate investment gaps to non-technical stakeholders — a
team that is strong on Protect and Detect but has never tested Recover is a very different risk profile
than one with the opposite gap, and CSF makes that gap visible and comparable across the industry.

## Common Pitfalls / Misconfigurations

- **Alert fatigue from untuned detections.** Shipping rules straight to production without tuning
  false-positive rates buries real signal in noise and trains analysts to dismiss alerts reflexively.
- **Treating Govern as paperwork.** Organizations that skip the Govern function in practice — no clear
  ownership, no communicated risk tolerance — tend to have inconsistent Protect/Detect/Respond
  implementations because no one is accountable for closing gaps between them.
- **No tested recovery process.** Many programs invest heavily in Detect and Respond but never
  rehearse Recover, discovering during a real ransomware incident that restoration procedures are
  undocumented or untested.
- **Detection content with no ownership or version history.** Ad hoc rules edited directly in a SIEM
  console, with no code review or rollback path, tend to rot — they silently break as log formats
  change, or accumulate unreviewed exceptions that quietly widen coverage gaps.
- **Incomplete log source coverage.** A gap in log collection (an unmonitored cloud service, an EDR
  agent that silently stopped reporting) creates a blind spot that no amount of SIEM tuning can
  compensate for, since detection can only run against data that was actually collected.

## Related TryHackMe Rooms in This Series

- [Offensive Security Intro](../offensive-security-intro/README.md)
- [The CIA Triad](../the-cia-triad/README.md)
- [Cryptography Concepts](../../fundamentals/cryptography-concepts/README.md)

## References

- NIST CSWP 29, *The NIST Cybersecurity Framework (CSF) 2.0*: https://nvlpubs.nist.gov/nistpubs/CSWP/NIST.CSWP.29.pdf
- NIST, *CSF 2.0 Reference Tool and Six Functions overview*: https://www.nist.gov/cyberframework
- NIST SP 800-61 Rev. 2, *Computer Security Incident Handling Guide*: https://csrc.nist.gov/pubs/sp/800/61/r2/final
- NIST SP 800-53 Rev. 5, *Security and Privacy Controls for Information Systems and Organizations*: https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final
- MITRE ATT&CK, *Enterprise Matrix*: https://attack.mitre.org/matrices/enterprise/
- Suricata, official project documentation: https://suricata.io/
- Snort, official project site: https://www.snort.org/
