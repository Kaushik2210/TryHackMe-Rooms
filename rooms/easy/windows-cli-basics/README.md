# Windows CLI Basics

**Platform:** TryHackMe · **Type:** Concept Guide (no personal run captured — see note below)
**Primary domain:** Windows / Operating Systems

> **Note:** This is a concept guide covering the material this TryHackMe room teaches, written from
> public documentation and reference material. It is not a personal walkthrough — no session evidence
> (screenshots, command output from a specific machine, room-specific answers) is included. Command
> examples below are generic reference examples, not captures from a completed session.

## Overview

Windows CLI Basics is TryHackMe's broad orientation room for working at a Windows command prompt at
all — the Windows-side counterpart to the platform's Linux CLI Basics room, continuing the same
story-driven, narrative approach on a new operating system. Its goal is orientation, not mastery: get a
newcomer comfortable opening a terminal, understanding *why* a command line exists alongside the GUI,
moving around the file system, and running a handful of everyday commands to inspect files and basic
system state. This is distinct from the **Windows Command Line** room, which is a focused, reference-
style deep dive into `cmd.exe` syntax, switches, and the command set itself as part of the Cyber
Security 101 path. Think of Windows CLI Basics as "how to drive," and Windows Command Line as "the
owner's manual for this specific engine." This guide covers the orientation material: what a shell is,
how to navigate with it, and the small set of commands a beginner needs to stop being intimidated by a
black window with a blinking cursor.

## Core Concepts

### Why a command line, when there's a GUI

Windows ships with more than one command-line environment — the legacy `cmd.exe` and the more capable
`PowerShell` — both reachable from the Start menu or via `Win + R` then typing `cmd` or `powershell`.
A command-line interface (CLI) trades the GUI's visual browsing for typed instructions, which is faster
for repetitive tasks, essential for scripting and automation, and often the only interface available on
a remote or headless system (e.g., a server accessed over SSH or RDP without a full desktop). Security
tooling and post-exploitation activity both live overwhelmingly at the command line, which is why
building basic comfort with it — regardless of which interpreter — is treated as a prerequisite skill.

### Opening a terminal and understanding the prompt

The default `cmd.exe` prompt shows the current working directory followed by `>`, e.g.:

```text
C:\Users\analyst>
```

Every command is typed after this prompt and executed with `Enter`. Commands can take **arguments**
(values the command acts on) and **switches**/**flags** (options starting with `/` in classic `cmd.exe`
syntax, e.g., `dir /a` to show hidden files) that modify behavior.

### Navigating the file system

| Command | Purpose |
|---|---|
| `dir` | List the contents of the current directory |
| `cd <path>` | Change directory (`cd ..` moves up one level, `cd \` goes to the drive root) |
| `cd` (no arguments) | Print the current working directory |
| `tree` | Display the directory structure as a tree |
| `<drive letter>:` | Switch to a different drive, e.g., typing `D:` and pressing Enter |

Unlike Linux's single `/` root, Windows paths are drive-letter based and use backslashes (`\`) as the
separator, e.g., `C:\Users\analyst\Documents`. Forward slashes generally still work in many contexts but
backslash is the native convention.

### Basic file and directory operations

```text
C:\Users\analyst> mkdir reports
C:\Users\analyst> cd reports
C:\Users\analyst\reports> echo Sample content > notes.txt
C:\Users\analyst\reports> type notes.txt
Sample content
C:\Users\analyst\reports> copy notes.txt notes-backup.txt
        1 file(s) copied.
C:\Users\analyst\reports> del notes-backup.txt
```

- `mkdir` (or `md`) creates a directory; `rmdir` (or `rd`) removes one.
- `type` prints a text file's contents to the screen — the rough equivalent of Linux's `cat`.
- `copy`, `move`, and `del` copy, move/rename, and delete files respectively.
- `echo` prints text, and with `>` redirects that output into a file (overwriting it), while `>>`
  appends instead.

### Getting basic system information

```text
C:\Users\analyst> whoami
desktop-01\analyst

C:\Users\analyst> hostname
desktop-01

C:\Users\analyst> ipconfig
Windows IP Configuration

Ethernet adapter Ethernet:
   IPv4 Address. . . . . . . . . . . : 192.0.2.15
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : 192.0.2.1
```

- `whoami` identifies the currently logged-in user (and, with `/priv` or `/groups`, their privileges
  and group memberships — details covered further in the dedicated Command Line room).
- `hostname` prints the machine name.
- `ipconfig` shows network adapter configuration; `ipconfig /all` gives a fuller picture including DNS
  servers and MAC addresses.

### Getting help

Every classic `cmd.exe` command supports a `/?` switch that prints its usage and available switches,
e.g., `dir /?`. This is the fastest way to learn a command's options without leaving the terminal, and
the habit transfers directly to later, more advanced rooms.

### Command history and productivity habits

`cmd.exe` retains a history of recently typed commands within the current session, navigable with the
Up and Down arrow keys, and `F7` opens a scrollable pop-up list of recent commands to re-select from.
Tab completion (pressing `Tab` after typing a partial file or folder name) fills in the rest of a path
automatically and is one of the single biggest time-savers for a beginner — it also reduces typos in
long paths, which matters because a single mistyped character sends a command to the wrong location
without necessarily producing an obvious error. Clearing the visible screen (without ending the session
or losing history) is done with `cls`, useful when a terminal window has filled with clutter from
previous output and a clean view is needed before running the next command.

### Wildcards

Both `dir` and file-manipulation commands like `copy`, `del`, and `move` accept wildcard characters to
match multiple files at once: `*` matches any sequence of characters, and `?` matches exactly one
character. For example, `dir *.txt` lists every file ending in `.txt` in the current directory, and
`del report?.log` would match `report1.log` and `report2.log` but not `report10.log`. Wildcards are a
small piece of syntax that pays off constantly once a user moves from single-file operations to batch
work, and they carry over conceptually (with different exact syntax) into PowerShell and Linux shells
alike.

## Why It Matters for Security

- **The CLI is the common denominator across remote access, scripting, and automation.** Whether an
  analyst is triaging a compromised host over a limited shell or an attacker is executing commands
  post-exploitation, comfort with basic navigation and file operations at a prompt is assumed baseline
  knowledge.
- **GUI-only investigation misses evidence.** Command-line tools can surface information (hidden files
  with `dir /a`, exact file timestamps, redirected output for logging) that is slower or impossible to
  gather purely through File Explorer.
- **It's the on-ramp to living-off-the-land techniques.** Attackers frequently rely on built-in CLI
  utilities already present on Windows rather than dropping custom malware, so recognizing normal CLI
  usage patterns helps distinguish it from abuse.

## Common Pitfalls / Misconfigurations

- **Confusing `cmd.exe` and PowerShell syntax.** Redirection, wildcards, and some commands behave
  differently between the two; mixing conventions produces confusing errors for beginners.
- **Forgetting that `del` has no recycle bin.** Files deleted from the command line typically bypass
  the Recycle Bin entirely, unlike deleting through File Explorer.
- **Not quoting paths with spaces.** A path like `C:\Program Files\App` needs quotes
  (`"C:\Program Files\App"`) or it gets parsed as two separate arguments.
- **Assuming `/?` and `-h`/`--help` are interchangeable.** Classic Windows commands overwhelmingly use
  the `/switch` convention, not the Unix-style single/double dash — a frequent source of "unknown
  option" errors for people coming from Linux.

## Related TryHackMe Rooms in This Series

1. [Windows Basics](../windows-basics/README.md)
2. Windows CLI Basics *(this room)*
3. [Windows Command Line](../windows-command-line/README.md)
4. [Windows PowerShell](../windows-powershell/README.md)
5. [Windows Fundamentals 1](../../fundamentals/windows-fundamentals-1/README.md)
6. [Windows Fundamentals 2](../../fundamentals/windows-fundamentals-2/README.md)
7. [Windows Fundamentals 3](../../fundamentals/windows-fundamentals-3/README.md)
8. [Linux CLI Basics](../linux-cli-basics/README.md)

## References

- [Microsoft Learn: Windows Commands (A-Z reference)](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/windows-commands)
- [Microsoft Learn: dir](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/dir)
- [Microsoft Learn: ipconfig](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/ipconfig)
- [Microsoft Learn: whoami](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/whoami)
- [Microsoft Learn: Command-line reference overview](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/command-line-reference-a-z)
