# Windows Basics

**Platform:** TryHackMe · **Type:** Concept Guide (no personal run captured — see note below)
**Primary domain:** Windows / Operating Systems

> **Note:** This is a concept guide covering the material this TryHackMe room teaches, written from
> public documentation and reference material. It is not a personal walkthrough — no session evidence
> (screenshots, command output from a specific machine, room-specific answers) is included. Command
> examples below are generic reference examples, not captures from a completed session.

## Overview

Windows Basics is TryHackMe's onboarding-style introduction to the Windows graphical environment,
aimed at people who have never had to operate a Windows desktop with intent — locating settings,
managing files, or checking whether the machine is secure. It frames the material as a first day at a
new job: log in, get oriented on the desktop and taskbar, open programs from the Start menu, create and
organize files in File Explorer, and take a first look at Task Manager and Windows Security. It sits
one level above the "Fundamentals" series in difficulty and scope, acting as a gentler, narrative-driven
entry point before the more systematic Windows Fundamentals 1–3 rooms. This guide covers the same
ground: desktop navigation, the file system as seen through the GUI, and the handful of built-in tools a
new user (or a security analyst working a Windows box for the first time) reaches for immediately.

## Core Concepts

### The desktop, taskbar, and Start menu

Windows presents its shell as a **desktop** — a background surface that can hold shortcuts — combined
with a **taskbar** anchored (by default) to the bottom of the screen. The taskbar hosts the **Start
menu** (the launcher for installed applications and a search box), pinned and running application
icons, and the **system tray**, which shows background processes such as network status, volume, and
the clock. The Start menu's search box is the fastest way to find both applications and system
utilities — typing a partial name (e.g., `task manager`, `control panel`, `cmd`) filters results
instantly, a habit worth building early because later rooms lean on it heavily to launch tools like
`services.msc` or `taskschd.msc`.

### File Explorer and the file system

**File Explorer** is the graphical file manager, reachable from the taskbar or via `Win + E`. It
presents storage as **drives** (`C:`, `D:`, and so on) rather than the single unified tree Linux uses —
each drive has its own root. Key locations a new user encounters:

| Location | Purpose |
|---|---|
| `C:\Users\<username>` | The signed-in user's home folder, containing `Desktop`, `Documents`, `Downloads`, `Pictures` |
| `C:\Program Files`, `C:\Program Files (x86)` | Installed 64-bit and 32-bit applications, respectively |
| `C:\Windows` | The operating system itself — system binaries, drivers, the registry hives |
| `C:\Windows\System32` | Core 64-bit system executables and DLLs; a directory beginners are warned not to modify carelessly |

File Explorer lets you create, rename, move, copy, and delete files and folders, and exposes basic
metadata (size, type, date modified) in its list and details views. Right-clicking a file surfaces a
**context menu** with options like "Properties," which opens a dialog showing attributes and — on an
NTFS volume — the **Security** tab, where file and folder permissions live (covered in more depth in
Windows Fundamentals 1).

### Managing windows and multitasking

Windows supports several built-in window-management shortcuts that speed up navigating multiple open
programs:

| Shortcut | Effect |
|---|---|
| `Alt + Tab` | Cycle between open windows |
| `Win + D` | Show/hide the desktop |
| `Win + Left` / `Win + Right` | Snap the active window to one half of the screen |
| `Win + L` | Lock the workstation |
| `Win + E` | Open File Explorer |

**Task View** (`Win + Tab`) shows all open windows and lets a user create multiple **virtual desktops**
to separate groups of work — for example, keeping a browser and notes on one desktop and a terminal on
another.

### Task Manager: a first look

**Task Manager** (`Ctrl + Shift + Esc`, or right-click the taskbar) is the primary built-in tool for
observing what is running and how the system is performing. Its tabs include:

- **Processes** — every running application and background process, with live CPU, memory, disk, and
  network usage columns. This is where an unresponsive application gets forcibly ended.
- **Performance** — real-time graphs for CPU, memory, disk, network, and (if present) GPU utilization.
- **Startup** (or **Startup apps**) — programs configured to launch automatically when a user logs in,
  along with a relative "startup impact" rating.
- **Users**, **Details**, and **Services** — more granular views used more heavily once a user moves
  into Windows Fundamentals 2's systems-administration material.

### A first look at Windows Security

**Windows Security** (searchable from the Start menu) is the consolidated dashboard for the built-in
protection features: Virus & threat protection (Microsoft Defender Antivirus), Firewall & network
protection, App & browser control (SmartScreen), and Device security. For a new user, the useful habit
is simply knowing this dashboard exists and glancing at it to confirm real-time protection is enabled
and no threats are flagged — the deeper mechanics of each pane are the subject of Windows Fundamentals 3.

### Personalization and accessibility basics

Beyond raw navigation, new users typically also encounter the **Settings → Personalization** and
**Settings → Accessibility** areas early on — changing desktop backgrounds and taskbar behavior in the
former, and adjusting text size, contrast themes, narrator, or magnifier options in the latter. These
settings are per-user rather than system-wide by default, which is a useful thing to know when
troubleshooting "it looks different on my account than my colleague's" reports on a shared machine, and
it reinforces the broader theme that most day-to-day Windows configuration lives at the user-profile
level rather than requiring administrative rights to change.

## Why It Matters for Security

- **The GUI is still the primary interface most users and many administrators operate through**, so
  understanding what "normal" looks like — which icons live in the system tray, what a legitimate UAC
  prompt looks like, which folders are writable by a standard user — is what lets an analyst or a
  suspicious end user notice when something is off.
- **File Explorer and Task Manager are first-response tools.** Before reaching for PowerShell or an
  EDR console, a huge amount of triage (what's running, what changed, what's in Downloads) happens
  through the same GUI covered here.
- **Familiarity reduces mistakes.** Confidently navigating drives, permissions dialogs, and the Windows
  Security dashboard prevents the kind of fumbling that either misses an indicator of compromise or
  accidentally makes a system change during an investigation.

## Common Pitfalls / Misconfigurations

- **Working directly in `C:\Windows\System32` or `C:\Program Files`** as if they were ordinary folders —
  moving or deleting files there can break installed software or the OS itself.
- **Ignoring the Startup tab in Task Manager.** Unwanted or malicious persistence frequently rides in as
  a low-impact startup entry that a new user never audits.
- **Treating Windows Security as "set and forget."** The dashboard reflects current state; it does not
  guarantee protection was never disabled or that a scan is current unless actually checked.
- **Confusing "closed window" with "closed process."** Some applications minimize to the system tray
  rather than terminate, which surprises users expecting Task Manager to show nothing running.

## Related TryHackMe Rooms in This Series

1. Windows Basics *(this room)*
2. [Windows Fundamentals 1](../../fundamentals/windows-fundamentals-1/README.md)
3. [Windows Fundamentals 2](../../fundamentals/windows-fundamentals-2/README.md)
4. [Windows Fundamentals 3](../../fundamentals/windows-fundamentals-3/README.md)
5. [Windows CLI Basics](../windows-cli-basics/README.md)
6. [Windows Command Line](../windows-command-line/README.md)
7. [Windows PowerShell](../windows-powershell/README.md)
8. [Active Directory Basics](../active-directory-basics/README.md)

## References

- [Microsoft Learn: Windows client documentation](https://learn.microsoft.com/en-us/windows/)
- [Microsoft Learn: Keyboard shortcuts in Windows](https://support.microsoft.com/en-us/windows/keyboard-shortcuts-in-windows-dcc61a57-8ff0-cffe-9796-cb9706c75eec)
- [Microsoft Learn: Task Manager overview](https://learn.microsoft.com/en-us/shows/inside/task-manager)
- [Microsoft Learn: Stay protected with Windows Security](https://support.microsoft.com/en-us/windows/stay-protected-with-windows-security-2ae0363d-0ada-c064-8b56-6a39afb6a963)
- [Microsoft Learn: NTFS overview](https://learn.microsoft.com/en-us/windows-server/storage/file-server/ntfs-overview)
