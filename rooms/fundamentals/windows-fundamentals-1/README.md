# Windows Fundamentals 1

**Platform:** TryHackMe · **Type:** Concept Guide (no personal run captured — see note below)
**Primary domain:** Windows / Operating Systems

> **Note:** This is a concept guide covering the material this TryHackMe room teaches, written from
> public documentation and reference material. It is not a personal walkthrough — no session evidence
> (screenshots, command output from a specific machine, room-specific answers) is included. Command
> examples below are generic reference examples, not captures from a completed session.

## Overview

Windows Fundamentals 1 is the first room in TryHackMe's three-part Windows Fundamentals series and
covers the desktop-facing basics: what Windows is and how its editions differ, the desktop/GUI layout,
the NTFS file system and the standard folder structure (including `System32`), and the account,
profile, and permission model that governs who can do what on a machine — capped off by User Account
Control (UAC), the mechanism that mediates privilege elevation. Where the standalone "Windows Basics"
room takes a narrative, task-driven approach to the same territory, Windows Fundamentals 1 is more
systematic and reference-oriented, laying groundwork that Windows Fundamentals 2 (system configuration
and administrative utilities) and Windows Fundamentals 3 (security tooling) build directly on top of.

## Core Concepts

### Windows editions and versioning

Windows ships in multiple editions per release (e.g., Home, Pro, Enterprise, Education for a given
version such as Windows 10 or 11), differentiated primarily by which features are licensed — Pro and
above add BitLocker, Group Policy, Remote Desktop hosting, and domain-join capability, none of which are
available on Home. Recognizing which edition a target is running is a routine early step in any
Windows-focused assessment, since it directly constrains which security controls and management
features can even be present.

### The desktop (GUI)

The desktop, taskbar, Start menu, and system tray form the primary graphical shell, with the **Settings**
app (introduced in Windows 8 and steadily absorbing capability since) and the legacy **Control Panel**
serving as the two main surfaces for changing system configuration. Control Panel remains the location
for a number of advanced or legacy settings not yet migrated to Settings, so competent Windows use
still requires knowing both exist and where to find each.

### The NTFS file system

Modern Windows uses **NTFS** (New Technology File System) as its default filesystem, superseding the
older FAT16/FAT32 and HPFS formats. NTFS adds, among other things:

- **Per-file/folder Access Control Lists (ACLs)** — fine-grained permissions (read, write, modify,
  full control, and more) assignable to specific users and groups, viewable and editable from a file's
  Properties → Security tab.
- **Journaling** — a transaction log that helps the filesystem recover to a consistent state after a
  crash or power loss.
- **Alternate Data Streams (ADS)** — the ability for a file to carry additional named data streams
  beyond its primary content, a legitimate feature (used, e.g., to tag downloaded files with their
  origin URL) that is also a well-known technique for hiding data or payloads.
- **Encryption and compression** attributes settable per file or folder (EFS — Encrypting File System —
  for the former).

### Standard folder structure and System32

| Path | Purpose |
|---|---|
| `C:\Users\<name>` | Per-user profile: `Desktop`, `Documents`, `Downloads`, `AppData` |
| `C:\Program Files` / `C:\Program Files (x86)` | Installed 64-bit / 32-bit applications |
| `C:\Windows` | The OS installation itself |
| `C:\Windows\System32` | Core 64-bit system binaries, DLLs, and drivers essential to Windows operation |
| `C:\Windows\SysWOW64` | 32-bit system binaries, present on 64-bit Windows for compatibility |

`System32` is a common target for both legitimate administration and malicious activity precisely
because so many trusted, digitally signed executables live there — attackers frequently rely on
binaries already present in this folder to blend in ("living off the land") rather than dropping new
executables that antivirus might flag.

### User accounts, profiles, and permissions

Every Windows account is backed by a unique **Security Identifier (SID)**, which is what the OS actually
uses internally to track ownership and permissions — the username is just a friendly label. Accounts
belong to local or domain **groups** (`Administrators`, `Users`, `Guests`, etc.), and group membership
is what typically grants or restricts capability, rather than permissions being assigned to individual
users one at a time. A user's **profile** (`C:\Users\<name>`, backed by the `ntuser.dat` registry hive)
stores their personal settings, files, and environment separately from other users on the same machine.

### User Account Control (UAC)

**UAC** is the mechanism that prevents software (and users) from silently making system-level changes
while logged in as, or as a member of, the local Administrators group. When an action requires elevated
privileges, Windows presents a consent prompt — for a standard user this requires entering administrator
credentials, while for an administrator it's typically a simple Yes/No confirmation (unless UAC's
notification level has been turned down). A shield icon overlaid on a program's icon or a button
indicates that launching it will trigger a UAC prompt. UAC operates by running processes with two
possible access tokens for an administrator account — a filtered, standard-user token used by default,
and a full administrative token only invoked after consent — which is what actually enforces the
"least privilege by default" behavior even for admin-level accounts.

### Local Security Policy basics

The **Local Security Policy** console (`secpol.msc`, available on Pro and above) exposes a subset of the
same policy categories that Group Policy manages at domain scale — account lockout thresholds, password
complexity requirements, audit policy, and user rights assignment (which local groups or accounts may,
for example, log on locally or access the machine from the network). On a standalone or workgroup
machine this is the primary place such baseline hardening settings are configured; on a domain-joined
machine, most of these settings are typically overridden by Group Policy pushed down from a domain
controller, and the local console mainly reflects the currently effective (already-merged) policy rather
than an independently authoritative one.

## Why It Matters for Security

- **NTFS permissions and SIDs are the actual enforcement mechanism behind "who can do what."**
  Understanding ACLs is a prerequisite for reasoning about privilege escalation via misconfigured file
  or service permissions later in more advanced material.
- **UAC is a mitigation, not an absolute boundary.** Numerous well-documented "UAC bypass" techniques
  exploit auto-elevating trusted binaries or DLL search-order weaknesses — recognizing how UAC is
  *supposed* to work is what makes those bypasses comprehensible.
- **System32's role as a trust anchor is exactly what makes it valuable to attackers.** Techniques that
  abuse legitimately signed binaries there (a subset of living-off-the-land binaries, or "LOLBins") are
  a staple of modern evasion.

## Common Pitfalls / Misconfigurations

- **Running daily activity as a local administrator account.** This defeats much of UAC's protective
  value, since an admin's consent prompt is trivially clicked through compared to a standard user
  needing actual credentials.
- **Overly permissive NTFS ACLs on sensitive folders** (e.g., granting `Everyone: Modify` on a directory
  containing service binaries or scripts) creates a straightforward privilege-escalation path.
- **Assuming file extension equals file type.** Windows Explorer hides known extensions by default,
  and a file's actual type is determined by its content/header, not its name — a long-standing social
  engineering and malware-delivery trick.
- **Disabling UAC notifications entirely** ("Never notify") to avoid prompts, which removes a real
  layer of protection against silent privilege escalation by malicious software.

## Related TryHackMe Rooms in This Series

1. Windows Fundamentals 1 *(this room)*
2. [Windows Fundamentals 2](../windows-fundamentals-2/README.md)
3. [Windows Fundamentals 3](../windows-fundamentals-3/README.md)
4. [Windows Basics](../../easy/windows-basics/README.md)
5. [Windows CLI Basics](../../easy/windows-cli-basics/README.md)
6. [Windows Command Line](../../easy/windows-command-line/README.md)
7. [Windows PowerShell](../../easy/windows-powershell/README.md)
8. [Active Directory Basics](../../easy/active-directory-basics/README.md)

## References

- [Microsoft Learn: NTFS overview](https://learn.microsoft.com/en-us/windows-server/storage/file-server/ntfs-overview)
- [Microsoft Learn: How User Account Control works](https://learn.microsoft.com/en-us/windows/security/application-security/application-control/user-account-control/how-it-works)
- [Microsoft Learn: Access control lists](https://learn.microsoft.com/en-us/windows/win32/secauthz/access-control-lists)
- [Microsoft Learn: Security identifiers](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/understand-security-identifiers)
- [Microsoft Learn: Alternate data streams](https://learn.microsoft.com/en-us/sysinternals/downloads/streams)
- [Microsoft Learn: Windows editions comparison](https://learn.microsoft.com/en-us/windows/whats-new/windows-11-editions)
