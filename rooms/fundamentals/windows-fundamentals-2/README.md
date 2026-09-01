# Windows Fundamentals 2

**Platform:** TryHackMe · **Type:** Concept Guide (no personal run captured — see note below)
**Primary domain:** Windows / Operating Systems

> **Note:** This is a concept guide covering the material this TryHackMe room teaches, written from
> public documentation and reference material. It is not a personal walkthrough — no session evidence
> (screenshots, command output from a specific machine, room-specific answers) is included. Command
> examples below are generic reference examples, not captures from a completed session.

## Overview

Windows Fundamentals 2 is the middle room in the three-part series, moving from Part 1's desktop and
file-permission material into the built-in utilities used to configure, troubleshoot, and inspect a
Windows system: System Configuration (`msconfig`), Computer Management, System Information, Resource
Monitor, the Command Prompt as an administrative surface, Task Scheduler, and — the centerpiece — the
Windows Registry. The throughline is "how does an administrator actually change and observe system
state," which makes this room the direct bridge between Part 1's conceptual permission model and Part
3's security-tooling focus. This guide covers the same utilities with an emphasis on what each one is
for and why it matters when investigating or hardening a machine.

## Core Concepts

### System Configuration (msconfig)

**System Configuration**, launched by running `msconfig`, is aimed at diagnosing startup problems. Its
tabs let an administrator choose between Normal, Diagnostic, and Selective startup; toggle which
services and startup items load; and adjust boot options (e.g., Safe Boot). It's explicitly a
troubleshooting tool rather than a day-to-day configuration surface — its Services and Startup tabs
have been substantially superseded by the richer views in Task Manager and Settings on modern Windows,
but it remains useful for isolating whether a problem is caused by a third-party service or driver.

### Computer Management

**Computer Management** (`compmgmt.msc`) bundles several administrative snap-ins into one console,
organized into three sections:

| Section | Contains |
|---|---|
| System Tools | Task Scheduler, Event Viewer, Shared Folders, Local Users and Groups, Device Manager |
| Storage | Disk Management (partitioning, volume management) |
| Services and Applications | Services console, WMI Control |

It's frequently the fastest single place to reach several otherwise separately-launched consoles,
which is why it's a common early stop during manual Windows administration or investigation.

### System Information and Resource Monitor

**System Information** (`msinfo32`) provides a static, detailed snapshot of hardware and software
configuration — BIOS version, installed hotfixes, running services, startup programs, and more — useful
for baselining a machine or gathering environment details quickly. **Resource Monitor** (`resmon`), by
contrast, is a live view, breaking down CPU, memory, disk, and network activity per process in far more
granular detail than Task Manager, including which process holds a given network connection or which
process is generating disk I/O on a specific file — useful when Task Manager's aggregate view isn't
precise enough to pin down a problem.

### Task Scheduler

**Task Scheduler** (`taskschd.msc`) lets tasks (running an application, a script, sending an email, and
more) be configured to execute automatically based on a **trigger** — at logon, at a scheduled time, on
system idle, or in response to a specific event log entry. Every scheduled task has an **action** (what
to run), one or more **triggers** (when to run it), and a **security context** (which account it runs
as, and whether it runs whether or not a user is logged in). This flexibility is exactly why Task
Scheduler is one of the most common built-in persistence mechanisms observed in real intrusions.

### The Command Prompt as an administrative tool

Part 2 revisits `cmd.exe` specifically in its administrative role — running it "as Administrator" (via
right-click or `Win + X`) to unlock commands and operations that require elevation, distinct from the
general orientation covered in the Windows CLI Basics and Windows Command Line rooms. An elevated prompt
is what's needed for many `sc`, `net`, and `reg` operations that touch system-wide state rather than a
single user's own resources.

### The Windows Registry

The **Registry** is a hierarchical database storing configuration for the OS, installed applications,
and per-user settings, organized into root keys called **hives**:

| Hive | Contains |
|---|---|
| `HKEY_LOCAL_MACHINE` (HKLM) | System-wide configuration — hardware, installed software, services |
| `HKEY_CURRENT_USER` (HKCU) | Settings for the currently logged-in user |
| `HKEY_USERS` (HKU) | Loaded profiles for all users, `HKCU` is effectively a pointer into this for the active user |
| `HKEY_CLASSES_ROOT` (HKCR) | File association and COM registration data |
| `HKEY_CURRENT_CONFIG` (HKCC) | Current hardware profile |

Each hive contains **keys** (folder-like containers) holding **values** of specific types (`REG_SZ` for
strings, `REG_DWORD` for 32-bit integers, `REG_BINARY`, `REG_MULTI_SZ`, and others). The GUI editor is
`regedit`; from the command line, `reg query`, `reg add`, and `reg delete` read and modify keys and
values directly, e.g.:

```text
C:\> reg query "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run"

HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
    SecurityHealth    REG_SZ    %windir%\system32\SecurityHealthSystray.exe
```

The `...\CurrentVersion\Run` and `RunOnce` keys (present in both `HKLM` and `HKCU`) are the most
well-known autostart locations — anything listed there launches automatically at logon, making them a
first stop for both legitimate startup configuration and malware persistence checks.

### Event Viewer

**Event Viewer** (`eventvwr.msc`), reachable through Computer Management or on its own, reads the
structured **Windows Event Log**, organized into channels such as Application, Security, System, and a
large set of application- and feature-specific operational logs under "Applications and Services Logs."
Each entry carries an Event ID, a source, a level (Information, Warning, Error, Critical), and a
timestamp, and can be filtered or exported for further analysis. The Security log in particular records
logon/logoff activity, privilege use, and object access when auditing is configured for them — making it
one of the most consulted logs during incident response, alongside Sysmon logs where that optional
tool has been deployed.

### Local Users and Groups

The **Local Users and Groups** snap-in (`lusrmgr.msc`, part of Computer Management's System Tools
section) manages accounts and groups scoped to the local machine, as distinct from domain accounts
managed centrally through Active Directory (covered in the Active Directory Basics room). From here an
administrator can create or disable local accounts, reset local passwords, and add or remove members
from local groups such as `Administrators` and `Remote Desktop Users` — a frequent target for both
legitimate local administration and, when abused, for establishing a persistent local backdoor account
on a single compromised host.

## Why It Matters for Security

- **Task Scheduler and Registry Run keys are two of the most common persistence mechanisms** documented
  across real-world intrusions, precisely because they're legitimate, everyday administrative features
  that blend in with normal system behavior.
- **The Registry is a primary artifact source in forensics and incident response** — it records
  installed software, USB device history, recently accessed files, and far more, all queryable without
  needing the original application present.
- **Resource Monitor and Computer Management provide ground-truth, per-process visibility** that's
  often the fastest way to confirm which process actually owns a suspicious network connection or file
  handle during live triage.

## Common Pitfalls / Misconfigurations

- **Editing the Registry without a backup.** `regedit` has no built-in undo; an incorrect key deletion
  or value change can render the OS or an application unusable. Exporting the affected key first is
  standard practice.
- **Ignoring non-obvious autostart locations.** Persistence isn't limited to the `Run` key — Scheduled
  Tasks, Services, the Startup folder, and WMI event subscriptions are equally common and are often
  missed by a Registry-only review.
- **Granting broad permissions on scheduled tasks or their target scripts.** A task running as SYSTEM
  that executes a script in a folder writable by standard users is a direct privilege-escalation path.
- **Treating `msconfig`'s Services/Startup tabs as authoritative.** They present a simplified view;
  Task Manager's Startup tab and Autoruns (a Sysinternals tool) give a more complete and accurate
  picture on modern Windows.

## Related TryHackMe Rooms in This Series

1. [Windows Fundamentals 1](../windows-fundamentals-1/README.md)
2. Windows Fundamentals 2 *(this room)*
3. [Windows Fundamentals 3](../windows-fundamentals-3/README.md)
4. [Windows Basics](../../easy/windows-basics/README.md)
5. [Windows CLI Basics](../../easy/windows-cli-basics/README.md)
6. [Windows Command Line](../../easy/windows-command-line/README.md)
7. [Windows PowerShell](../../easy/windows-powershell/README.md)
8. [Active Directory Basics](../../easy/active-directory-basics/README.md)

## References

- [Microsoft Learn: Windows Registry information for advanced users](https://learn.microsoft.com/en-us/troubleshoot/windows-server/performance/windows-registry-advanced-users)
- [Microsoft Learn: Registry hives](https://learn.microsoft.com/en-us/windows/win32/sysinfo/registry-hives)
- [Microsoft Learn: Task Scheduler start page](https://learn.microsoft.com/en-us/windows/win32/taskschd/task-scheduler-start-page)
- [Microsoft Learn: reg command reference](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/reg)
- [Microsoft Learn: System Configuration utility troubleshooting](https://learn.microsoft.com/en-us/troubleshoot/windows-client/deployment/system-configuration-utility-troubleshoot-startup)
- [Microsoft Sysinternals: Autoruns](https://learn.microsoft.com/en-us/sysinternals/downloads/autoruns)
