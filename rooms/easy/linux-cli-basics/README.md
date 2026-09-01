# Linux CLI Basics

**Platform:** TryHackMe · **Type:** Concept Guide (no personal run captured — see note below)
**Primary domain:** Linux / Operating Systems

> **Note:** This is a concept guide covering the material this TryHackMe room teaches, written from
> public documentation and reference material. It is not a personal walkthrough — no session evidence
> (screenshots, command output from a specific machine, room-specific answers) is included. Command
> examples below are generic reference examples, not captures from a completed session.

## Overview

The command-line interface (CLI) is the primary way security practitioners interact with Linux
systems — over SSH, in a CTF terminal, or on a compromised host during an engagement, there is
frequently no GUI available. This guide introduces the absolute basics of working in a Linux shell:
what a prompt and working directory are, how to move through the filesystem, how to inspect and
manipulate files, and how to get help without leaving the terminal. It assumes no prior command-line
experience and sets up the vocabulary used throughout the rest of this series.

## Core Concepts

### The prompt and the current working directory

A typical Linux shell prompt shows the logged-in user, hostname, and current directory, e.g.
`alice@server:~$`. The shell always executes relative to a **current working directory**, which you can
print with `pwd` (print working directory):

```text
$ pwd
/home/alice
```

### Navigating the filesystem

| Command | Purpose |
|---|---|
| `pwd` | Print the current working directory |
| `ls` | List directory contents |
| `ls -la` | List all entries (including hidden dotfiles) in long format |
| `cd <dir>` | Change directory |
| `cd ..` | Move up one level |
| `cd ~` or `cd` | Return to the home directory |
| `cd -` | Return to the previous directory |

```text
$ ls -la
total 24
drwxr-xr-x  4 alice alice 4096 Sep  1 10:00 .
drwxr-xr-x  6 root  root  4096 Sep  1 09:00 ..
-rw-r--r--  1 alice alice  220 Sep  1 09:00 .bash_logout
drwxr-xr-x  2 alice alice 4096 Sep  1 09:00 Documents
```

Paths can be **absolute** (start at `/`, the filesystem root — e.g. `/home/alice/Documents`) or
**relative** to the current directory (e.g. `Documents`, `../shared`). `.` refers to the current
directory and `..` to its parent — used constantly when moving files or restricting scripts.

### Inspecting and creating files

| Command | Purpose |
|---|---|
| `cat file` | Print a file's full contents to standard output |
| `less file` | Page through a file's contents interactively |
| `head -n 10 file` | Print the first 10 lines |
| `tail -n 10 file` | Print the last 10 lines |
| `touch file` | Create an empty file, or update its modification timestamp |
| `mkdir dir` | Create a directory (`-p` to create intermediate parents) |
| `cp src dst` | Copy a file or (`-r`) directory |
| `mv src dst` | Move or rename a file/directory |
| `rm file` | Delete a file (`-r` for directories, `-f` to skip confirmation) |

```text
$ echo "hello" > greeting.txt
$ cat greeting.txt
hello
$ cp greeting.txt backup.txt
$ mv backup.txt archive/backup.txt
```

`>` overwrites a file with a command's output; `>>` appends instead. This is standard shell
**redirection**, distinct from the `|` pipe operator, which feeds one command's output into another's
input (e.g. `cat access.log | grep 404`).

### Searching for content and files

`grep` searches text for a pattern; `find` searches the filesystem for files matching criteria:

```text
$ grep -i "error" application.log
$ grep -rn "TODO" src/
$ find / -name "*.conf" -type f 2>/dev/null
```

- `grep -i` — case-insensitive; `-r` — recursive; `-n` — show line numbers.
- `find <path> -name <pattern> -type f` — search `<path>` for files (`-type f`) matching a glob.

### Getting help without leaving the terminal

Every standard Linux command ships a manual page. `man <command>` opens it; `<command> --help` prints a
shorter usage summary; `whatis <command>` gives a one-line description.

```text
$ man ls
$ ls --help
$ whatis grep
grep (1)             - print lines that match patterns
```

### Users and identity

`whoami` prints the current effective username; `id` shows the full UID/GID and group membership:

```text
$ whoami
alice
$ id
uid=1000(alice) gid=1000(alice) groups=1000(alice),27(sudo)
```

### Wildcards and glob patterns

The shell expands unquoted wildcard characters against matching filenames *before* running a command —
this is called **globbing**, and it happens in the shell itself, not in the program being called:

```text
$ ls *.txt
greeting.txt  notes.txt
$ rm backup-2024-*.log
$ cp ./configs/*.conf ./backup/
```

`*` matches any sequence of characters, `?` matches exactly one, and `[abc]`/`[a-z]` matches any single
character in the given set/range. Because expansion happens before the command runs, `rm *` is
particularly dangerous — the shell can expand it to far more files than the operator intended if run in
the wrong directory.

### Standard streams and exit status

Every process has three standard I/O streams: **stdin** (0, input), **stdout** (1, normal output), and
**stderr** (2, error/diagnostic output) — kept separate so that output and errors can be redirected
independently:

```text
$ find / -name "*.conf" 2>/dev/null       # discard stderr, keep stdout
$ command > out.txt 2> err.txt            # split streams to separate files
$ command > all.txt 2>&1                  # merge stderr into stdout, then redirect
```

Every command also returns a numeric **exit status** on completion — `0` for success, non-zero for
failure — retrievable immediately afterward via the `$?` variable, and this is what `&&`/`||` chaining
tests:

```text
$ grep -q "root" /etc/passwd
$ echo $?
0
```

### Archiving and compression

Moving or backing up multiple files commonly uses `tar` (tape archive, bundles files/directories into
one archive) often combined with compression:

```text
$ tar -czf backup.tar.gz Documents/     # create (c), gzip (z), file (f)
$ tar -xzf backup.tar.gz                # extract (x)
$ tar -tzf backup.tar.gz                # list (t) contents without extracting
```

`zip`/`unzip` are also common, particularly for cross-platform archives shared with Windows users.

## Why It Matters for Security

- **The CLI is often the only interface available.** SSH sessions, reverse shells, and many CTF/exam
  environments provide a bare terminal with no GUI, so fluency here is a prerequisite for essentially
  everything downstream in offensive and defensive security work.
- **Enumeration starts with navigation and search.** `find`, `grep`, and directory traversal are the
  building blocks of both legitimate system administration and post-exploitation reconnaissance
  (locating configuration files, credentials, or writable paths).
- **Redirection and piping compose tools.** Chaining small utilities together (`cat | grep | sort`) is
  the idiomatic Linux approach and underlies more advanced log analysis and scripting covered later.

## Common Pitfalls / Misconfigurations

- **Using `rm -rf` carelessly**, especially with a wrong or unset path variable — a classic
  destructive mistake with no undo on most filesystems.
- **Confusing absolute and relative paths** inside scripts, causing them to behave differently
  depending on the directory they're launched from.
- **Ignoring hidden files** (`ls` without `-a`) and missing dotfiles like `.bash_history`, `.ssh/`, or
  `.env`, which frequently contain credentials or configuration relevant to an assessment.
- **Not quoting variables/filenames with spaces or special characters**, which can cause a command to
  be parsed differently than intended — a frequent source of both bugs and, in scripts handling
  untrusted input, injection issues.

## Related TryHackMe Rooms in This Series

1. [Operating Systems: Introduction](../operating-systems-introduction/README.md)
2. [Operating System Security](../operating-system-security/README.md)
3. Linux CLI Basics *(this room)*
4. [Linux Fundamentals Part 1](../../fundamentals/linux-fundamentals-part-1/README.md)
5. [Linux Fundamentals Part 2](../../fundamentals/linux-fundamentals-part-2/README.md)
6. [Linux Fundamentals Part 3](../../fundamentals/linux-fundamentals-part-3/README.md)
7. [Linux Shells](../linux-shells/README.md)

## References

- [GNU Coreutils Manual](https://www.gnu.org/software/coreutils/manual/coreutils.html)
- [ls(1) — Linux man page](https://man7.org/linux/man-pages/man1/ls.1.html)
- [cd — POSIX shell built-in](https://pubs.opengroup.org/onlinepubs/9699919799/utilities/cd.html)
- [grep(1) — Linux man page](https://man7.org/linux/man-pages/man1/grep.1.html)
- [find(1) — Linux man page](https://man7.org/linux/man-pages/man1/find.1.html)
- [man(1) — Linux man page](https://man7.org/linux/man-pages/man1/man.1.html)
- [Bash Reference Manual — Redirections](https://www.gnu.org/software/bash/manual/html_node/Redirections.html)
- [Bash Reference Manual — Filename Expansion (globbing)](https://www.gnu.org/software/bash/manual/html_node/Filename-Expansion.html)
- [tar(1) — Linux man page](https://man7.org/linux/man-pages/man1/tar.1.html)
- [Advanced Bash-Scripting Guide — Exit Status (TLDP)](https://tldp.org/LDP/abs/html/exit-status.html)
