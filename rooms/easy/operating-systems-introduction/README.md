# Operating Systems: Introduction

**Platform:** TryHackMe · **Type:** Concept Guide (no personal run captured — see note below)
**Primary domain:** Operating Systems

> **Note:** This is a concept guide covering the material this TryHackMe room teaches, written from
> public documentation and reference material. It is not a personal walkthrough — no session evidence
> (screenshots, command output from a specific machine, room-specific answers) is included. Command
> examples below are generic reference examples, not captures from a completed session.

## Overview

An operating system (OS) is the software layer that sits between raw hardware and the applications a
user runs, arbitrating access to the CPU, memory, storage, and peripherals so that many programs can
share one machine safely. Before any command-line work, packet capture, or exploit development makes
sense, it helps to understand what the OS is actually doing: scheduling processes, isolating memory
spaces, mediating file and device access, and enforcing the permission boundaries that later become the
target of privilege-escalation techniques. This guide surveys the major desktop and server operating
system families — Windows, Linux/Unix, and macOS — and the core OS concepts (kernel, shell, process,
filesystem) that every later room in this series builds on.

## Core Concepts

### What an operating system actually does

At minimum, every general-purpose OS provides four services:

| Function | Description |
|---|---|
| Process management | Creates, schedules, and terminates running programs; each process gets its own virtual memory space and a scheduling slice of CPU time. |
| Memory management | Maps virtual addresses used by programs to physical RAM, and swaps data to disk when RAM is exhausted. |
| File system management | Organizes persistent storage into a hierarchy of files and directories, and enforces read/write/execute permissions on them. |
| Device / I/O management | Provides a uniform interface (drivers) so software doesn't need to know the specifics of each disk controller, network card, or GPU. |

### Kernel vs. shell vs. userland

- **Kernel** — the privileged core of the OS. It runs in a CPU protection ring with direct hardware
  access (ring 0 on x86) and exposes system calls (`open()`, `read()`, `fork()`, `execve()`, and so on)
  that user programs invoke to request services.
- **Shell** — a program (not part of the kernel) that provides a command interface — text-based
  (`bash`, `zsh`, `cmd.exe`, `PowerShell`) or graphical — for a human or script to instruct the OS.
- **Userland** — everything that runs in unprivileged mode: applications, system utilities, libraries.
  A crash in userland should not be able to crash the kernel; this separation is the basis of OS
  stability and, from a security standpoint, of privilege separation.

### Major OS families

**Windows** (NT kernel lineage) dominates enterprise desktops and a large share of servers. It uses the
NTFS filesystem, the Windows Registry for configuration, and a security model built on Security
Identifiers (SIDs), Access Control Lists (ACLs), and user/group tokens.

**Linux** is a kernel, not a full OS by itself — a "Linux distribution" (Ubuntu, Debian, Fedora, Arch,
Kali, etc.) pairs the Linux kernel with GNU userland tools, a package manager, and often a desktop
environment. Linux underpins the majority of internet-facing servers, cloud infrastructure, container
images, and embedded/IoT devices, which is why it is the primary focus of the rest of this series.

**macOS** is built on Darwin, a BSD- and Mach-kernel-derived Unix core, wrapped in Apple's proprietary
GUI and frameworks. Being POSIX-compliant under the hood, many Linux command-line skills transfer
directly to a macOS terminal.

### Processes and the process table

Every running program is a **process** with a unique Process ID (PID), an owner (user/group), a parent
process (PPID), and its own memory space. On Linux, `ps` and `top` expose the live process table:

```text
$ ps -ef | head -5
UID          PID    PPID  C STIME TTY          TIME CMD
root           1       0  0 09:12 ?        00:00:02 /sbin/init
root         512       1  0 09:12 ?        00:00:00 /usr/sbin/sshd -D
alice       1830    1790  0 09:41 pts/0    00:00:00 -bash
alice       1902    1830  0 09:52 pts/0    00:00:00 ps -ef
```

- `-e` selects every process; `-f` gives the "full" format including PPID and start time.
- `UID`/PID/PPID chain tells you ownership and lineage — important later when reasoning about which
  user context a service or exploited process is running under.

### Filesystem hierarchy (Linux/Unix)

Unix-like systems present a single unified tree rooted at `/`, unlike Windows' per-drive letters. Key
top-level directories, per the [Filesystem Hierarchy Standard (FHS)](https://refspecs.linuxfoundation.org/FHS_3.0/fhs-3.0.html):

| Path | Purpose |
|---|---|
| `/bin`, `/usr/bin` | User executable binaries |
| `/sbin`, `/usr/sbin` | System administration binaries (historically requiring root) |
| `/etc` | Host-specific system configuration files |
| `/home` | Per-user home directories |
| `/root` | Home directory for the root user |
| `/var` | Variable data — logs (`/var/log`), spool files, caches |
| `/tmp` | World-writable temporary storage, typically cleared on reboot |
| `/dev` | Device nodes (e.g., `/dev/sda` for a disk) |
| `/proc`, `/sys` | Virtual filesystems exposing kernel and process state as files |

## Why It Matters for Security

Understanding the OS layer is the prerequisite for everything downstream in offensive and defensive
security work:

- **Privilege boundaries.** Exploits and misconfigurations are meaningful because the OS enforces a
  boundary between unprivileged users and root/SYSTEM. You cannot reason about privilege escalation
  without first understanding what "privilege" means at the OS level (UID 0, SIDs, capability bits).
- **Process and memory concepts underlie exploitation.** Buffer overflows, injection, and many
  post-exploitation techniques operate on the process/memory model described above.
- **The filesystem is the primary evidence and attack surface.** Log locations (`/var/log`),
  configuration files (`/etc`), and permission bits are all filesystem-level constructs that both
  attackers and defenders manipulate or inspect.
- **Cross-platform awareness.** Real environments are heterogeneous — an assessment or CTF room may
  span Windows, Linux, and macOS hosts, so recognizing each OS's conventions (paths, permission models,
  service managers) speeds up enumeration and reduces mistakes.

## Common Pitfalls / Misconfigurations

- **Treating "the OS" as monolithic.** Confusing kernel-level behavior with shell or application
  behavior leads to wrong conclusions during troubleshooting or exploitation (e.g., assuming a
  privilege issue is a "bash bug" when it's actually a `setuid` bit on a binary).
- **Ignoring the process owner.** A service running as root that accepts unsanitized input is a much
  higher-value target than the same bug in a process running as an unprivileged user — the `UID`
  column in `ps -ef` is not decorative.
- **Assuming Windows and Linux permission models are equivalent.** NTFS ACLs and POSIX permission bits
  are conceptually related but not directly translatable; conflating them causes errors when moving
  between platforms.
- **Not distinguishing `/tmp` from persistent storage.** Placing anything security-sensitive (or
  anything you need to survive a reboot) in a world-writable, frequently-cleared directory like `/tmp`
  is a common newcomer mistake.

## Related TryHackMe Rooms in This Series

1. Operating Systems: Introduction *(this room)*
2. [Operating System Security](../operating-system-security/README.md)
3. [Linux CLI Basics](../linux-cli-basics/README.md)
4. [Linux Fundamentals Part 1](../../fundamentals/linux-fundamentals-part-1/README.md)
5. [Linux Fundamentals Part 2](../../fundamentals/linux-fundamentals-part-2/README.md)
6. [Linux Fundamentals Part 3](../../fundamentals/linux-fundamentals-part-3/README.md)
7. [Linux Shells](../linux-shells/README.md)

## References

- [Filesystem Hierarchy Standard (FHS) 3.0](https://refspecs.linuxfoundation.org/FHS_3.0/fhs-3.0.html)
- [ps(1) — Linux man page](https://man7.org/linux/man-pages/man1/ps.1.html)
- [proc(5) — Linux man page](https://man7.org/linux/man-pages/man5/proc.5.html)
- [The Linux Kernel Archives](https://www.kernel.org/)
- [Microsoft Learn: Windows architecture](https://learn.microsoft.com/en-us/windows-hardware/drivers/gettingstarted/windows-architecture)
- [Apple Developer: Darwin / Kernel and Device Drivers](https://developer.apple.com/library/archive/documentation/Darwin/Conceptual/KernelProgramming/Architecture/Architecture.html)
