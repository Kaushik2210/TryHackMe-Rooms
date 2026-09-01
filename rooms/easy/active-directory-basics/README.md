# Active Directory Basics

**Platform:** TryHackMe · **Type:** Concept Guide (no personal run captured — see note below)
**Primary domain:** Windows / Operating Systems / Identity

> **Note:** This is a concept guide covering the material this TryHackMe room teaches, written from
> public documentation and reference material. It is not a personal walkthrough — no session evidence
> (screenshots, command output from a specific machine, room-specific answers) is included. Command
> examples below are generic reference examples, not captures from a completed session.

## Overview

Active Directory Basics introduces Microsoft's directory service — the technology that underlies
identity and access management for the large majority of enterprise Windows networks. Framed around
administering a small fictional company's network, the room covers the core structural building blocks
(domains, organizational units, users and groups), how administrative control is delegated and applied
at scale through Group Policy, and the authentication protocols — chiefly Kerberos, with a secondary
look at NetNTLM — that let a domain-joined machine trust a central directory rather than maintaining its
own local account database. It builds directly on the Windows Fundamentals series' account and
permission concepts, extending them from a single machine to an entire network of them under centralized
management.

## Core Concepts

### What Active Directory is

**Active Directory Domain Services (AD DS)** is a hierarchical database and set of services that
centralizes identity (user and computer accounts), authentication, and policy enforcement across a
network of Windows machines. Rather than each computer maintaining its own local user list, domain-
joined machines defer authentication and authorization decisions to one or more **Domain Controllers
(DCs)** — servers that hold a writable copy of the directory and run the services (Kerberos Key
Distribution Center, LDAP, DNS integration) that make the domain function.

### Domains, trees, and forests

| Term | Definition |
|---|---|
| Domain | A administrative and security boundary — a collection of objects (users, computers, groups) sharing one directory database, e.g., `corp.example.com` |
| Tree | One or more domains sharing a contiguous DNS namespace (`corp.example.com`, `sales.corp.example.com`) |
| Forest | The top-level container — one or more trees, sharing a common schema and Global Catalog, but not necessarily a contiguous namespace |
| Trust | A relationship allowing users in one domain to authenticate to resources in another; trusts can be one-way or two-way, and transitive or non-transitive |

A single domain is sufficient for most small-to-medium organizations; trees and forests, along with
inter-forest trusts, become relevant as organizations merge, acquire other companies, or need to
segment administrative boundaries.

### Organizational Units (OUs)

**Organizational Units** are containers used to arrange users, computers, and groups into a logical
structure that mirrors organizational needs — commonly by department, location, or device role (e.g.,
separate `Workstations` and `Servers` OUs, or `Sales`, `Marketing`, and `Management` OUs for users).
OUs exist primarily to serve two purposes: applying **Group Policy** at a targeted scope, and enabling
**delegation** — granting a specific user or group administrative rights over just that OU (e.g., the
ability to reset passwords for the Sales OU) without making them a full Domain Administrator.

### Users, groups, and computers

Every domain-joined identity — user, computer, or service — is represented as an object in the
directory, uniquely identified internally by a **Security Identifier (SID)**, mirroring the local-account
model covered in Windows Fundamentals 1 but issued and tracked centrally by the domain. **Security
groups** are the primary mechanism for assigning permissions at scale: rather than granting rights to
individual accounts, rights are granted to a group, and users are added to or removed from that group as
their role changes. Built-in high-privilege groups like `Domain Admins` and `Enterprise Admins` warrant
particular scrutiny, since membership in them typically grants control over the entire domain or forest.

### Group Policy Objects (GPOs)

A **Group Policy Object** is a collection of configuration settings — security options, software
deployment, scripts, registry-based preferences, and more — that gets applied to the users and/or
computers within the OU (or domain, or site) it's linked to. GPOs let an administrator, for example,
enforce a password complexity policy, deploy a printer connection, or restrict which applications can
run, uniformly across every machine in an OU, without touching each machine individually. Policy
application generally follows an inheritance order often summarized as **Local → Site → Domain → OU**
(commonly abbreviated LSDOU), with settings closer to the object in that chain taking precedence unless
a GPO is explicitly configured to enforce/override.

### Authentication: Kerberos

**Kerberos** is the default authentication protocol for any reasonably current Active Directory domain,
replacing the older, weaker NTLM family for most interactive logons. At a conceptual level, the flow is:

1. A user authenticates to the **Key Distribution Center (KDC)**, a service running on every Domain
   Controller, and receives a **Ticket Granting Ticket (TGT)** — proof of authentication, encrypted with
   a key derived from the KDC's own secret (specifically, the `krbtgt` account's password hash).
2. When the user needs to access a specific service (a file share, a database, a web application), they
   present their TGT back to the KDC and request a **service ticket** for that specific resource.
2. The service ticket is encrypted using a key derived from the target service account's own credential,
   so only that service can decrypt and validate it.
3. The user presents the service ticket directly to the target service, which validates it locally
   without needing to contact the Domain Controller again for that request.

This ticket-based design means a domain controller isn't queried for every single resource access, only
for the initial TGT and each new service ticket request — an efficiency and scalability property, on
top of Kerberos's mutual-authentication and replay-resistance properties.

### Authentication: NetNTLM (briefly)

**NetNTLM** is the legacy challenge-response authentication protocol still supported for backward
compatibility — used when Kerberos isn't available (e.g., authenticating by IP address rather than
hostname, or against non-domain-joined resources). It's weaker than Kerberos in several well-understood
ways and its continued presence on a network is generally treated as something to minimize rather than
rely on.

## Why It Matters for Security

- **Active Directory is the single highest-value target in most enterprise networks.** Compromising a
  Domain Admin-equivalent account or the `krbtgt` account effectively means compromising every
  domain-joined resource, which is why AD compromise is the objective of the overwhelming majority of
  enterprise ransomware and targeted-intrusion playbooks.
- **Kerberos's design, while strong, has known abuse patterns at a conceptual level** — most notably
  **Kerberoasting**, where an attacker requests service tickets for accounts running services (which are
  encrypted with that service account's password-derived key) and then attempts to crack the ticket
  offline, entirely without triggering a failed-logon event on the DC.
- **Delegation and GPOs are powerful, so misconfiguring either directly translates to privilege
  escalation paths** — a poorly scoped OU delegation or a GPO with a misconfigured ACL are both common,
  realistic routes from a low-privilege domain user to broader control.

## Common Pitfalls / Misconfigurations

- **Over-provisioning `Domain Admins` membership.** Every additional account in a high-privilege group
  is another potential path to full domain compromise; least-privilege delegation via OUs exists
  specifically to avoid this.
- **Service accounts with weak or never-rotated passwords.** Because Kerberoasting works entirely
  offline once a service ticket is obtained, a weak service account password is crackable with no
  further interaction with the domain controller.
- **Flat OU structures with no delegation boundaries**, forcing routine administrative tasks (like
  password resets) to go through overly privileged accounts rather than scoped delegation.
- **Leaving legacy NTLM enabled network-wide** "just in case," when it could be restricted or audited,
  needlessly preserving a weaker fallback authentication path.
- **GPOs linked with unintended inheritance or missing "Enforced" settings**, causing security baselines
  to silently not apply to sub-OUs where administrators assumed they did.

## Related TryHackMe Rooms in This Series

1. [Windows Basics](../windows-basics/README.md)
2. [Windows Fundamentals 1](../../fundamentals/windows-fundamentals-1/README.md)
3. [Windows Fundamentals 2](../../fundamentals/windows-fundamentals-2/README.md)
4. [Windows Fundamentals 3](../../fundamentals/windows-fundamentals-3/README.md)
5. [Windows CLI Basics](../windows-cli-basics/README.md)
6. [Windows Command Line](../windows-command-line/README.md)
7. [Windows PowerShell](../windows-powershell/README.md)
8. Active Directory Basics *(this room)*

## References

- [Microsoft Learn: Active Directory Domain Services overview](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview)
- [Microsoft Learn: Organizational Units](https://learn.microsoft.com/en-us/windows/win32/ad/organizational-units)
- [Microsoft Learn: Group Policy overview](https://learn.microsoft.com/en-us/windows/win32/group-policy/group-policy-start-page)
- [Microsoft Learn: Kerberos authentication overview](https://learn.microsoft.com/en-us/windows-server/security/kerberos/kerberos-authentication-overview)
- [Microsoft Learn: How the Kerberos Version 5 authentication protocol works](https://learn.microsoft.com/en-us/windows-server/security/kerberos/kerberos-authentication-overview)
- [Microsoft Learn: Security identifiers](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/understand-security-identifiers)
- [Microsoft Learn: Active Directory security groups](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/understand-security-groups)
