# Windows Command Line

**Platform:** TryHackMe · **Type:** Concept Guide (no personal run captured — see note below)
**Primary domain:** Windows / Operating Systems

> **Note:** This is a concept guide covering the material this TryHackMe room teaches, written from
> public documentation and reference material. It is not a personal walkthrough — no session evidence
> (screenshots, command output from a specific machine, room-specific answers) is included. Command
> examples below are generic reference examples, not captures from a completed session.

## Overview

Windows Command Line is part of TryHackMe's Cyber Security 101 learning path and is a focused,
reference-style deep dive into `cmd.exe` — the default command interpreter shipped with every version of
Windows since the NT lineage replaced the MS-DOS-era `COMMAND.COM`. Where **Windows CLI Basics** is
about general orientation (opening a terminal, not panicking, moving between directories), this room
assumes that comfort already exists and goes deeper into `cmd.exe`'s actual command syntax: file and
directory manipulation with switches, process and service inspection, user and network administration
commands, and the batch-scripting constructs that let `cmd.exe` commands be chained into repeatable
scripts. This guide treats `cmd.exe` as a syntax and capability reference — the commands a defender or
analyst is expected to recognize instantly when they show up in a process tree or a script.

## Core Concepts

### cmd.exe fundamentals

`cmd.exe` interprets typed lines as either **internal commands** (built directly into the interpreter,
like `dir`, `cd`, `copy`) or **external commands** (standalone `.exe`/`.bat`/`.cmd` files on the `PATH`,
like `ipconfig.exe` or `tasklist.exe`). Command separators allow chaining on one line:

| Separator | Behavior |
|---|---|
| `&` | Run the next command regardless of the first command's exit code |
| `&&` | Run the next command only if the previous one succeeded (exit code 0) |
| `\|\|` | Run the next command only if the previous one failed |
| `\|` | Pipe: send one command's output as the next command's input |

### File and directory manipulation

| Command | Purpose |
|---|---|
| `dir /a` | List files including hidden and system files |
| `dir /s` | List recursively through subdirectories |
| `attrib` | View or change file attributes (`+h` hidden, `+r` read-only, `+s` system) |
| `xcopy` / `robocopy` | Copy files/directories with far more control than `copy` — recursion, retry logic, mirroring |
| `findstr` | Search for text patterns inside files, roughly analogous to `grep` |
| `fc` | Compare the contents of two files |

```text
C:\logs> findstr /i "error" app.log
2026-08-30 14:02:11 ERROR Connection refused
2026-08-30 14:05:44 error Timeout waiting for response
```

`findstr /i` makes the search case-insensitive; it also supports regular-expression-style matching with
`/r`.

### Process and service inspection

| Command | Purpose |
|---|---|
| `tasklist` | List currently running processes, with PID, memory usage, and session |
| `taskkill /PID <pid>` or `/IM <name>` | Terminate a process by PID or image name (`/F` forces it) |
| `sc query` | Query the state of a Windows service |
| `net start` / `net stop <service>` | Start or stop a service by name |

```text
C:\> tasklist /FI "IMAGENAME eq notepad.exe"

Image Name                     PID Session Name        Session#    Mem Usage
========================= ======== ================ =========== ============
notepad.exe                   4820 Console                    1     11,240 K
```

`/FI` applies a filter — here restricting output to a single image name — which is far more useful than
scrolling a full process list when scripting or triaging.

### User and network administration commands

| Command | Purpose |
|---|---|
| `net user` | List local users, or view/create/modify a specific account |
| `net localgroup` | List or manage local groups (e.g., add a user to `Administrators`) |
| `net share` | List or configure SMB shares |
| `ipconfig /all` | Full network adapter configuration including DNS and MAC |
| `netstat -ano` | Active TCP/UDP connections and listening ports, with owning PID |
| `nslookup` | Query DNS records for a hostname |

```text
C:\> net user
User accounts for \\DESKTOP-01
-------------------------------------------------------------------------
Administrator            analyst                  DefaultAccount
Guest                    WDAGUtilityAccount
The command completed successfully.
```

`net user <username>` with no further arguments shows details for a single account (group memberships,
password policy flags); appending a password and switches can create or modify one — an action that
requires administrative privileges.

### Batch scripting basics

`cmd.exe` commands can be saved into a `.bat` file and executed as a script, using:

```bat
@echo off
REM Simple batch script example
set TARGET=C:\reports
if not exist %TARGET% mkdir %TARGET%
for %%f in (*.log) do echo Found log file: %%f
```

- `@echo off` suppresses echoing each command before it runs.
- `set` defines an environment-style variable, referenced with `%VARNAME%`.
- `if exist` / `if not exist` performs conditional file checks.
- `for %%f in (...)` iterates over a set of files — a construct commonly abused in malicious batch
  scripts to enumerate and act on files in bulk.

### Environment variables and PATH

`cmd.exe` exposes system and user environment variables via `%VARNAME%` syntax; `set` with no arguments
lists them all. `PATH` in particular controls which directories are searched, in order, when an external
command is typed without a full path — a detail relevant to both normal troubleshooting ("why isn't this
tool found") and to DLL/PATH-hijacking style attacks.

### Redirection in more detail

Beyond the basic `>` and `>>` seen with `echo`, `cmd.exe` supports redirecting each of a program's
standard streams independently: standard output (`1`), standard error (`2`), and standard input (`0`).
`command > output.txt 2>&1` redirects both standard output and standard error into the same file — the
`2>&1` syntax specifically means "point stream 2 at wherever stream 1 currently points," and the order
matters; writing it before the `>` redirection would send errors to the console instead. `command <
input.txt` feeds a file's contents in as standard input. This matters when scripting: a command that
appears to succeed interactively can still fail silently in a batch job if its error stream was never
captured, which is a common cause of "it worked when I ran it by hand" support tickets.

### Comparing to PowerShell

Every command covered in this room has a rough PowerShell equivalent — `tasklist` maps to
`Get-Process`, `net user` maps to Active-Directory or local-account cmdlets, `findstr` maps to
`Select-String` — but the two are not drop-in replacements for each other. `cmd.exe` commands operate on
plain text streams, while PowerShell's cmdlets operate on structured objects, which is why PowerShell
output can be filtered and manipulated with far more precision. Recognizing which interpreter a given
line of shell activity in a log or process tree belongs to is itself a useful triage skill, since the
two leave different artifacts and are favored for different purposes by both administrators and
attackers.

## Why It Matters for Security

- **`cmd.exe` is one of the most heavily used living-off-the-land binaries.** Malware and attackers
  routinely spawn `cmd.exe /c <command>` to blend in with legitimate administrative activity, so
  recognizing normal versus suspicious command syntax is a core detection skill.
- **Batch scripts are a common persistence and delivery mechanism.** Scheduled tasks, startup folders,
  and Group Policy logon scripts frequently invoke `.bat` files, making the syntax here directly
  relevant to incident response and threat hunting.
- **`netstat`, `tasklist`, and `net` commands are first-line triage tools** for identifying suspicious
  connections, unexpected processes, or unauthorized accounts on a host without needing third-party
  tooling.

## Common Pitfalls / Misconfigurations

- **Running `taskkill /F` indiscriminately.** Force-killing the wrong process (or a parent process with
  dependent children) can destabilize the system or lose forensic context during an investigation.
- **Storing secrets in plaintext batch scripts.** `set PASSWORD=...` in a `.bat` file is visible to
  anyone who can read the file, and often ends up in `netstat`/process-argument logging as well.
- **Misreading `&` vs `&&`.** Assuming a chained command only runs "on success" when a bare `&` was
  used (which runs regardless) is a frequent scripting logic error.
- **Overlooking `PATH` order.** A malicious binary placed earlier in a writable `PATH` directory can be
  executed instead of the intended legitimate tool of the same name.

## Related TryHackMe Rooms in This Series

1. [Windows Basics](../windows-basics/README.md)
2. [Windows CLI Basics](../windows-cli-basics/README.md)
3. Windows Command Line *(this room)*
4. [Windows PowerShell](../windows-powershell/README.md)
5. [Windows Fundamentals 1](../../fundamentals/windows-fundamentals-1/README.md)
6. [Windows Fundamentals 2](../../fundamentals/windows-fundamentals-2/README.md)
7. [Windows Fundamentals 3](../../fundamentals/windows-fundamentals-3/README.md)

## References

- [Microsoft Learn: Command-line reference A-Z](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/windows-commands)
- [Microsoft Learn: tasklist](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/tasklist)
- [Microsoft Learn: taskkill](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/taskkill)
- [Microsoft Learn: netstat](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/netstat)
- [Microsoft Learn: net user](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/net-user)
- [Microsoft Learn: findstr](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/findstr)
- [Microsoft Learn: for](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/for)
