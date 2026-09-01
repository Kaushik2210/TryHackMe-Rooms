# Windows PowerShell

**Platform:** TryHackMe · **Type:** Concept Guide (no personal run captured — see note below)
**Primary domain:** Windows / Operating Systems

> **Note:** This is a concept guide covering the material this TryHackMe room teaches, written from
> public documentation and reference material. It is not a personal walkthrough — no session evidence
> (screenshots, command output from a specific machine, room-specific answers) is included. Command
> examples below are generic reference examples, not captures from a completed session.

## Overview

Windows PowerShell, also part of TryHackMe's Cyber Security 101 path, introduces the second (and now
primary) command-line shell built into Windows. Unlike `cmd.exe`, which the earlier Windows Command Line
room covers as a text-stream interpreter, PowerShell is an object-oriented shell and scripting language:
commands return structured .NET objects rather than plain text, which can be filtered, sorted, and piped
between commands with far more precision. The room walks through PowerShell's verb-noun **cmdlet**
naming convention, core navigation and file-system cmdlets, the help system, and basic system- and
network-information gathering — building toward the recognition that PowerShell is both a legitimate
administrative powerhouse and one of the most common tools abused in modern intrusions. This guide
covers that same ground: cmdlet syntax, the pipeline, and the everyday commands a defender needs to read
fluently.

## Core Concepts

### Cmdlets and verb-noun syntax

PowerShell commands are called **cmdlets** (pronounced "command-lets") and follow a consistent
`Verb-Noun` naming pattern, e.g., `Get-Process`, `Stop-Service`, `New-Item`. Standard verbs (`Get`,
`Set`, `New`, `Remove`, `Start`, `Stop`) make behavior predictable even for cmdlets you've never used
before. Running `Get-Verb` lists the approved verbs Microsoft defines for PowerShell module authors.

### The help system

```text
PS C:\> Get-Help Get-Process

NAME
    Get-Process

SYNTAX
    Get-Process [[-Name] <string[]>] [-ComputerName <string[]>] ...

PS C:\> Get-Help Get-Process -Examples
PS C:\> Get-Help Get-Process -Full
```

`Get-Help <cmdlet>` is the built-in reference; `-Examples` shows sample usage and `-Full` includes every
parameter and detailed remarks. The first time PowerShell's help is used, it may prompt to run
`Update-Help` to download the latest documentation set, since help content isn't fully bundled offline
by default.

### Discovering commands: Get-Command and aliases

`Get-Command` lists available cmdlets, functions, and aliases, and can be filtered:

```text
PS C:\> Get-Command -Noun Process

CommandType     Name                    Version    Source
-----------     ----                    -------    ------
Function        Debug-Process           7.0.0.0    Microsoft.PowerShell.Management
Cmdlet          Get-Process             7.0.0.0    Microsoft.PowerShell.Management
Cmdlet          Stop-Process            7.0.0.0    Microsoft.PowerShell.Management
```

PowerShell also ships built-in **aliases** for familiar commands from both `cmd.exe` and Unix shells —
`dir` and `ls` both resolve to `Get-ChildItem`, `cd` to `Set-Location`, `cat` to `Get-Content`, `ps` to
`Get-Process`. `Get-Alias` lists all defined aliases and what they map to.

### Navigating the file system

| Cmdlet | Common alias | Purpose |
|---|---|---|
| `Get-Location` | `pwd` | Print current directory |
| `Set-Location` | `cd` | Change directory |
| `Get-ChildItem` | `ls`, `dir` | List directory contents |
| `Get-Content` | `cat`, `type` | Read a file's contents |
| `New-Item` | — | Create a file or directory (`-ItemType Directory`/`File`) |
| `Copy-Item`, `Move-Item`, `Remove-Item` | — | Copy, move/rename, delete files or directories |

Because PowerShell can address more than the local filesystem — the registry, certificate store, and
environment variables are all exposed as navigable **PSDrives** (e.g., `HKLM:`, `Cert:`, `Env:`) — the
same `Get-ChildItem`/`Set-Location` verbs work across all of them, a design detail that distinguishes it
sharply from `cmd.exe`.

### The pipeline and object filtering

Because cmdlets pass structured objects (not plain text) down the pipeline, downstream cmdlets can
select, filter, and sort on actual object properties:

```text
PS C:\> Get-Process | Where-Object { $_.CPU -gt 100 } | Sort-Object CPU -Descending | Select-Object -First 5 Name, CPU

Name                    CPU
----                    ---
chrome                 342.5
explorer               210.1
svchost                155.8
```

- `Where-Object` filters objects by a condition against `$_` (the current pipeline object).
- `Sort-Object` orders by one or more properties.
- `Select-Object` picks specific properties and/or limits the number of results (`-First`, `-Last`).
- `Format-Table` / `Format-List` control display, and are usually implicit at the end of a pipeline.

### System and network information

```text
PS C:\> Get-Service | Where-Object { $_.Status -eq 'Running' } | Select-Object -First 3

Status   Name               DisplayName
------   ----               -----------
Running  BITS               Background Intelligent Transfer Service
Running  Dhcp               DHCP Client
Running  Dnscache           DNS Client

PS C:\> Get-NetIPAddress -AddressFamily IPv4 | Select-Object InterfaceAlias, IPAddress
PS C:\> Get-CimInstance Win32_OperatingSystem | Select-Object Caption, Version, OSArchitecture
```

`Get-Service`, `Get-Process`, `Get-NetIPAddress`, and `Get-CimInstance` (the modern replacement for the
older WMI-based `Get-WmiObject`) form the backbone of PowerShell-based system enumeration, used
constantly in both administration and security tooling.

### Execution policy

PowerShell restricts script execution by default through the **execution policy**, checked with
`Get-ExecutionPolicy` and set with `Set-ExecutionPolicy` (e.g., `RemoteSigned`, which allows local
scripts to run but requires downloaded scripts to be digitally signed). It's important to understand
this is a safety guardrail against accidental execution, not a hard security boundary — it can be
bypassed in numerous well-documented ways (`-ExecutionPolicy Bypass` on invocation, for instance), so it
should never be relied on as the sole control against malicious script execution.

### Variables and simple scripting

PowerShell variables are prefixed with `$` and are loosely typed by default (`$name = "analyst"`), but
can be explicitly typed when needed (`[int]$count = 5`). Basic control flow — `if`/`elseif`/`else`,
`foreach`, `while`, and `for` loops — follows a C-like brace syntax familiar from many other languages,
which is part of why PowerShell scripting tends to be approachable for people coming from a general
programming background rather than only a shell-scripting one. A simple script saved as a `.ps1` file
and run directly demonstrates the same object-pipeline behavior available interactively:

```text
$processes = Get-Process | Where-Object { $_.WorkingSet -gt 100MB }
foreach ($p in $processes) {
    Write-Output "$($p.Name) is using $([math]::Round($p.WorkingSet / 1MB)) MB"
}
```

Note the use of `100MB` — PowerShell natively understands byte-size unit suffixes (`KB`, `MB`, `GB`) as
numeric literals, sparing the author from manual unit conversion.

## Why It Matters for Security

- **PowerShell is the single most common "living-off-the-land" tool in modern Windows intrusions.**
  Its deep .NET integration lets it download payloads, interact with the registry and AD, and execute
  in-memory without touching disk, which is exactly why understanding its normal syntax and object
  pipeline is essential to spotting abuse.
- **The same cmdlets used for administration are used for enumeration by attackers** — `Get-Process`,
  `Get-Service`, `Get-CimInstance`, and Active-Directory cmdlets are equally at home in a sysadmin's
  script and in a post-exploitation reconnaissance script.
- **PowerShell logging (Script Block Logging, Module Logging, Transcription) is a major detection
  surface** that security teams enable specifically because the shell is so heavily used offensively;
  understanding cmdlet syntax is what makes those logs readable during an investigation.

## Common Pitfalls / Misconfigurations

- **Treating the execution policy as a security control.** It is trivially bypassed and should never be
  the only defense against malicious script execution — logging, AMSI, and application control matter
  far more.
- **Confusing aliases with the underlying cmdlet's actual parameters.** `ls -la` habits from Linux don't
  translate; `Get-ChildItem` uses different parameter names (`-Force` to show hidden items, for
  instance), and assuming alias parity across all switches leads to errors.
- **Piping formatted output instead of objects.** Running a cmdlet through `Format-Table` and then
  trying to pipe the result further breaks the pipeline, because formatting cmdlets produce display
  objects, not the original data objects — a frequent beginner mistake.
- **Leaving PowerShell logging disabled.** Without Script Block Logging enabled, defenders lose
  visibility into exactly what a PowerShell-based attack executed, even if the process launch itself was
  logged.

## Related TryHackMe Rooms in This Series

1. [Windows Basics](../windows-basics/README.md)
2. [Windows CLI Basics](../windows-cli-basics/README.md)
3. [Windows Command Line](../windows-command-line/README.md)
4. Windows PowerShell *(this room)*
5. [Windows Fundamentals 1](../../fundamentals/windows-fundamentals-1/README.md)
6. [Windows Fundamentals 2](../../fundamentals/windows-fundamentals-2/README.md)
7. [Windows Fundamentals 3](../../fundamentals/windows-fundamentals-3/README.md)
8. [Active Directory Basics](../active-directory-basics/README.md)

## References

- [Microsoft Learn: PowerShell documentation](https://learn.microsoft.com/en-us/powershell/scripting/overview)
- [Microsoft Learn: Get-Help](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/get-help)
- [Microsoft Learn: Get-Command](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/get-command)
- [Microsoft Learn: about_Pipelines](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_pipelines)
- [Microsoft Learn: Set-ExecutionPolicy](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.security/set-executionpolicy)
- [Microsoft Learn: about_Execution_Policies](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_execution_policies)
- [Microsoft Learn: PowerShell security features (Script Block Logging, AMSI)](https://learn.microsoft.com/en-us/powershell/scripting/windows-powershell/wmf/whats-new/script-logging)
