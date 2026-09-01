# Linux Fundamentals Part 2

**Platform:** TryHackMe · **Type:** Concept Guide (no personal run captured — see note below)
**Primary domain:** Linux / Operating Systems

> **Note:** This is a concept guide covering the material this TryHackMe room teaches, written from
> public documentation and reference material. It is not a personal walkthrough — no session evidence
> (screenshots, command output from a specific machine, room-specific answers) is included. Command
> examples below are generic reference examples, not captures from a completed session.

## Overview

Part 2 of TryHackMe's Linux Fundamentals module builds on Part 1's first commands by covering remote
access via SSH, deeper filesystem interaction (copying, moving, editing files), understanding command
flags through manual pages, and — most importantly for later security work — the Linux file permission
model. This guide covers that same progression: connecting to a remote host securely, working with
files beyond simple navigation, and reading/interpreting/changing permission bits.

## Core Concepts

### Remote access with SSH

SSH (Secure Shell) is the standard way to obtain an interactive shell on a remote Linux machine over an
encrypted channel, superseding legacy cleartext protocols like Telnet:

```text
$ ssh alice@10.10.10.10
alice@10.10.10.10's password:
alice@target:~$
```

Non-default ports and key-based authentication are common in practice:

```text
$ ssh -p 2222 alice@10.10.10.10
$ ssh -i ~/.ssh/id_ed25519 alice@10.10.10.10
```

SSH also supports secure file transfer without a separate protocol, via `scp` (or the more modern
`sftp`):

```text
$ scp report.txt alice@10.10.10.10:/home/alice/
$ scp alice@10.10.10.10:/home/alice/loot.txt ./
```

A fuller treatment of key-based authentication and hardening lives in the
[Operating System Security](../../easy/operating-system-security/README.md) guide.

### Reading manual pages and command flags

Almost every Linux command accepts flags (also called switches or options) that modify its behavior.
`man <command>` opens the full manual; the `SYNOPSIS` section shows accepted flag syntax, and flags can
usually be combined (`ls -la` = `ls -l -a`):

```text
$ man cp
NAME
       cp - copy files and directories
SYNOPSIS
       cp [OPTION]... SOURCE DEST
       cp [OPTION]... SOURCE... DIRECTORY
```

### File and directory manipulation

| Command | Purpose |
|---|---|
| `cp -r src/ dst/` | Copy a directory recursively |
| `mv old new` | Move or rename |
| `rm -r dir/` | Remove a directory and its contents |
| `mkdir -p a/b/c` | Create nested directories in one call |
| `nano file` / `vim file` | Edit a file's contents in a terminal text editor |
| `wc -l file` | Count lines in a file |
| `file somefile` | Identify a file's type from its contents (not just extension) |

### The Linux permission model in depth

`ls -l` reveals the permission string, owner, and group for every file:

```text
$ ls -l script.sh
-rwxr-xr-- 1 alice devs 512 Sep  1 10:00 script.sh
```

Reading left to right: file type, owner permissions (`rwx`), group permissions (`r-x`), other
permissions (`r--`), followed by owner (`alice`) and group (`devs`). Each triad independently controls
read/write/execute for that class of user. Permissions are changed with `chmod`, either symbolically or
numerically:

```text
$ chmod u+x script.sh          # add execute for the owner
$ chmod go-w script.sh         # remove write for group and others
$ chmod 750 script.sh          # owner rwx, group r-x, others ---
```

Octal values sum per triad: read=4, write=2, execute=1. `750` = owner `7` (rwx), group `5` (r-x),
others `0` (none). Ownership itself — who is "owner" and which group is attached — changes with
`chown` and `chgrp`:

```text
$ sudo chown bob script.sh
$ sudo chgrp devs script.sh
$ sudo chown bob:devs script.sh   # both in one call
```

### Key filesystem locations

Building on the FHS overview from the Operating Systems Introduction guide, Part 2 typically highlights
directories relevant to day-to-day use: `/home` for user data, `/etc` for system-wide configuration,
`/var/log` for logs, and `/etc/passwd` / `/etc/shadow` for account and credential metadata (the latter
readable only by root).

### Finding files and content system-wide

Beyond simple navigation, Part 2 typically introduces searching the whole filesystem rather than just
the current directory:

```text
$ find / -type f -name "*.bak" 2>/dev/null
$ find /home -type d -name "backup"
$ find / -perm -4000 -type f 2>/dev/null   # files with the SUID bit set
$ locate passwd                             # fast search using a prebuilt index (updatedb)
```

`find` walks the filesystem live and supports rich filters (`-type`, `-name`, `-perm`, `-mtime`,
`-size`); `locate` is much faster but relies on a periodically-updated index (`updatedb`), so it can
miss very recent changes.

### Disk usage and free space

```text
$ df -h              # filesystem-level free/used space, human-readable
$ du -sh Documents/   # summarized size of a directory tree
$ du -h --max-depth=1 /var | sort -rh | head
```

`df` reports per-mounted-filesystem space; `du` reports actual space consumed by files/directories,
which is useful for tracking down what is filling a disk.

### Symbolic and hard links

`ln` creates references to existing files. A **symbolic link** (`ln -s`) is a small file that points to
a path by name — it can cross filesystems and points to nothing valid if the target is removed. A
**hard link** (`ln` without `-s`) is a second directory entry pointing to the same underlying inode on
the same filesystem — the file's data isn't actually removed until every hard link to it is deleted.

```text
$ ln -s /var/log/nginx/access.log ~/nginx.log
$ ls -l ~/nginx.log
lrwxrwxrwx 1 alice alice 27 Sep  1 10:00 nginx.log -> /var/log/nginx/access.log
```

## Why It Matters for Security

- **SSH is the dominant remote-management protocol**, and understanding its authentication flow (and
  where credentials or keys are stored) is essential before any lateral-movement or persistence
  discussion later in a broader curriculum.
- **Permission bits are the mechanism, not just a formality.** `chmod`/`chown` misuse — whether by an
  administrator or as the deliberate target of a lab exercise — is one of the most common
  privilege-escalation and information-disclosure root causes on Linux.
- **`/etc/passwd` vs `/etc/shadow` illustrates defense-in-depth**: account metadata is world-readable
  by design (usernames must be resolvable), but the actual password hashes are isolated in a file only
  root can read, limiting what an unprivileged compromise can immediately extract.

## Common Pitfalls / Misconfigurations

- **`chmod 777`/`chmod -R 777`** as a quick fix for a "permission denied" error — this grants
  read/write/execute to everyone and is one of the most common misconfigurations found in both CTFs
  and real audits.
- **Copying files with `cp` and losing intended permissions** — `cp` does not always preserve mode
  bits unless `-p` (preserve) is used, which can silently loosen or tighten access.
- **SSHing in as root directly** instead of using a named account plus `sudo`, which removes
  accountability (no per-user audit trail) and increases the blast radius of a stolen credential.
- **Editing `/etc/passwd` or `/etc/shadow` by hand** instead of using dedicated tools (`useradd`,
  `passwd`, `usermod`), risking a malformed file that locks out all logins.

## Related TryHackMe Rooms in This Series

1. [Operating Systems: Introduction](../../easy/operating-systems-introduction/README.md)
2. [Operating System Security](../../easy/operating-system-security/README.md)
3. [Linux CLI Basics](../../easy/linux-cli-basics/README.md)
4. [Linux Fundamentals Part 1](../linux-fundamentals-part-1/README.md)
5. Linux Fundamentals Part 2 *(this room)*
6. [Linux Fundamentals Part 3](../linux-fundamentals-part-3/README.md)
7. [Linux Shells](../../easy/linux-shells/README.md)

## References

- [ssh(1) — OpenSSH client](https://man.openbsd.org/ssh.1)
- [scp(1) — OpenSSH secure copy](https://man.openbsd.org/scp.1)
- [chmod(1) — Linux man page](https://man7.org/linux/man-pages/man1/chmod.1.html)
- [chown(1) — Linux man page](https://man7.org/linux/man-pages/man1/chown.1.html)
- [passwd(5) — Linux man page](https://man7.org/linux/man-pages/man5/passwd.5.html)
- [shadow(5) — Linux man page](https://man7.org/linux/man-pages/man5/shadow.5.html)
- [GNU Coreutils Manual](https://www.gnu.org/software/coreutils/manual/coreutils.html)
- [Filesystem Hierarchy Standard (FHS) 3.0](https://refspecs.linuxfoundation.org/FHS_3.0/fhs-3.0.html)
- [find(1) — Linux man page](https://man7.org/linux/man-pages/man1/find.1.html)
- [locate(1) / updatedb(8) — Linux man pages](https://man7.org/linux/man-pages/man1/locate.1.html)
- [df(1) — Linux man page](https://man7.org/linux/man-pages/man1/df.1.html)
- [du(1) — Linux man page](https://man7.org/linux/man-pages/man1/du.1.html)
- [ln(1) — Linux man page](https://man7.org/linux/man-pages/man1/ln.1.html)
