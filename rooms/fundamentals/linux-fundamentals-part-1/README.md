# Linux Fundamentals Part 1

**Platform:** TryHackMe · **Type:** Concept Guide (no personal run captured — see note below)
**Primary domain:** Linux / Operating Systems

> **Note:** This is a concept guide covering the material this TryHackMe room teaches, written from
> public documentation and reference material. It is not a personal walkthrough — no session evidence
> (screenshots, command output from a specific machine, room-specific answers) is included. Command
> examples below are generic reference examples, not captures from a completed session.

## Overview

This is the first room in TryHackMe's three-part Linux Fundamentals module, which progresses from
first commands (Part 1), through file management, permissions, and remote access (Part 2), to
automation, package management, and logging (Part 3). Part 1 establishes the absolute basics: a short
history of Linux, connecting to a remote lab machine, and the first handful of commands used to output
text, identify the current user, and move around the filesystem. This guide covers that same ground
from public reference material — the history of the OS family, the anatomy of a terminal session, and
core navigation/output commands — without reproducing TryHackMe's room text or task-specific flags.

## Core Concepts

### A brief history of Linux

Linux began in 1991 when Linus Torvalds released a hobby kernel for 386(486) AT-compatible computers,
built in the spirit of the free-software GNU Project started by Richard Stallman in 1983. Combining the
Linux kernel with GNU userland tools (compiler, shell, coreutils) produced a complete, freely
redistributable Unix-like operating system. Because both the kernel and the GNU tools are open source,
anyone can inspect, modify, and redistribute them — which is why hundreds of independent
**distributions** (Debian, Ubuntu, Fedora, Arch, and security-focused ones like Kali) exist, each
packaging the same underlying kernel with different defaults, package managers, and philosophies.

### Connecting to a Linux machine

In a lab or CTF context, you typically reach a target Linux machine one of two ways:

- A **web-based terminal / attack box** provided by the platform, which puts you directly at a shell
  prompt in the browser.
- **SSH** from your own machine, e.g. `ssh user@10.10.10.10`, which opens an encrypted remote shell
  session (SSH itself is covered in depth in Part 2 and in the OS Security concept guide).

Either way, you land at a shell prompt — commonly `bash` — ready to accept commands.

### First commands: output and identity

`echo` prints text (or the expanded value of a variable) to standard output:

```text
$ echo "TryHackMe"
TryHackMe
$ echo $HOME
/home/alice
```

`whoami` reports the current effective username — useful for confirming who you are after an `su` or
`sudo` context switch:

```text
$ whoami
alice
```

### Navigating the filesystem

The core navigation trio, expanded on in the [Linux CLI Basics](../../easy/linux-cli-basics/README.md)
guide, is:

| Command | Purpose |
|---|---|
| `pwd` | Print the current working directory |
| `ls` | List the contents of a directory |
| `cd <path>` | Change into another directory |

```text
$ pwd
/home/alice
$ ls
Documents  Downloads  notes.txt
$ cd Documents
$ pwd
/home/alice/Documents
```

`ls` accepts flags that change what it shows: `-l` for a long listing with permissions, owner, size,
and modification time; `-a` to include hidden dotfiles; these compose as `ls -la`.

### A first look at searching text: `grep`

Part 1 material typically introduces `grep` just enough to search a file for a specific string or
pattern, setting up its fuller treatment (recursive search, regex, flags) in Part 3:

```text
$ grep "ERROR" access.log
$ grep -i "error" access.log      # case-insensitive
```

### Chaining commands

The shell lets you combine multiple commands on one line:

- `cmd1 && cmd2` — run `cmd2` only if `cmd1` succeeds (exit status 0).
- `cmd1 ; cmd2` — run both regardless of `cmd1`'s outcome.
- `cmd1 | cmd2` — pipe `cmd1`'s output into `cmd2`'s input.

```text
$ mkdir project && cd project
$ cat notes.txt | grep "TODO"
```

### Tab completion and command history

Two features make the shell far faster to use than typing every character manually:

- **Tab completion** — pressing `Tab` while typing a command, path, or (for many programs) argument
  auto-completes it, or lists possibilities on a second press if ambiguous. This reduces typos and
  speeds up navigation of deep directory trees considerably.
- **Command history** — `bash` remembers previously entered commands; the Up/Down arrows cycle through
  them, and `Ctrl+R` opens a reverse incremental search over history. The `history` command lists it,
  and it persists between sessions in `~/.bash_history`.

```text
$ cd Doc<Tab>
$ cd Documents/
$ history | tail -5
```

### Clearing the screen and stopping a running command

- `clear` (or `Ctrl+L`) clears the terminal display without affecting history or running processes.
- `Ctrl+C` sends an interrupt signal (`SIGINT`) to the foreground process, stopping most commands.
- `Ctrl+D` sends an end-of-file signal, commonly used to close an interactive shell or input stream.

### The `sudo` command, briefly

Some commands and files require elevated privileges. `sudo` executes a single command as another user
(root by default), prompting for the *invoking user's own* password rather than root's:

```text
$ sudo apt update
[sudo] password for alice:
```

A full treatment of the permission model behind why this is necessary appears in
[Operating System Security](../../easy/operating-system-security/README.md) and
[Linux Fundamentals Part 2](../linux-fundamentals-part-2/README.md).

### Command-line arguments and options at a glance

Most Linux commands follow a loose convention distinguishing three kinds of input on a command line:

```text
$ command [options] [arguments]
$ ls -l /home/alice
      ^   ^
   option  argument (path)
```

Short options are usually a single dash plus one letter (`-l`), often combinable (`-la`); long options
use a double dash and a full word (`--all`), which is easier to read in scripts but slower to type
interactively. Not every command follows this convention exactly, which is precisely why `man` and
`--help` are worth checking rather than guessing.

## Why It Matters for Security

- **Every later technique assumes CLI fluency.** Enumeration scripts, exploitation frameworks, and
  post-exploitation tooling are all driven from a shell; Part 1's commands are the literal vocabulary
  used in every subsequent room in this series.
- **Knowing your identity context matters immediately.** `whoami`/`id` become the first commands run
  after landing a shell on a target, to establish what privilege level has been obtained.
- **The history and distribution landscape informs targeting.** Recognizing which distribution and
  version a target runs (via `/etc/os-release`, banner grabs, or package versions) shapes which known
  vulnerabilities and default configurations are relevant.

## Common Pitfalls / Misconfigurations

- **Treating all Linux distributions as identical.** Package manager (`apt` vs. `dnf` vs. `pacman`),
  default shell, and file layout can differ meaningfully between distributions.
- **Not checking command exit status.** Chaining with `&&` assumes the previous command succeeded;
  silently ignoring failures (e.g. with `;`) can mask errors in scripts.
- **Copy-pasting commands without understanding flags.** `rm -rf`, `chmod 777`, and similar
  commands are commonly pasted from tutorials without understanding their scope, leading to data loss
  or unintended world-writable permissions.

## Related TryHackMe Rooms in This Series

1. [Operating Systems: Introduction](../../easy/operating-systems-introduction/README.md)
2. [Operating System Security](../../easy/operating-system-security/README.md)
3. [Linux CLI Basics](../../easy/linux-cli-basics/README.md)
4. Linux Fundamentals Part 1 *(this room)*
5. [Linux Fundamentals Part 2](../linux-fundamentals-part-2/README.md)
6. [Linux Fundamentals Part 3](../linux-fundamentals-part-3/README.md)
7. [Linux Shells](../../easy/linux-shells/README.md)

## References

- [GNU Coreutils Manual](https://www.gnu.org/software/coreutils/manual/coreutils.html)
- [echo — Bash Reference Manual (builtin commands)](https://www.gnu.org/software/bash/manual/html_node/Bourne-Shell-Builtins.html)
- [ls(1) — Linux man page](https://man7.org/linux/man-pages/man1/ls.1.html)
- [cd — POSIX shell built-in](https://pubs.opengroup.org/onlinepubs/9699919799/utilities/cd.html)
- [grep(1) — Linux man page](https://man7.org/linux/man-pages/man1/grep.1.html)
- [whoami(1) — Linux man page](https://man7.org/linux/man-pages/man1/whoami.1.html)
- [The Linux Kernel Archives — history](https://www.kernel.org/category/about.html)
- [GNU Project — About](https://www.gnu.org/gnu/about-gnu.html)
- [Bash Reference Manual — Command History](https://www.gnu.org/software/bash/manual/html_node/Bash-History-Facilities.html)
- [Bash Reference Manual — Programmable Completion](https://www.gnu.org/software/bash/manual/html_node/Programmable-Completion.html)
- [sudo(8) — Linux man page](https://man7.org/linux/man-pages/man8/sudo.8.html)
- [signal(7) — Linux man page](https://man7.org/linux/man-pages/man7/signal.7.html)
