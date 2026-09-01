# Linux Fundamentals Part 3

**Platform:** TryHackMe · **Type:** Concept Guide (no personal run captured — see note below)
**Primary domain:** Linux / Operating Systems

> **Note:** This is a concept guide covering the material this TryHackMe room teaches, written from
> public documentation and reference material. It is not a personal walkthrough — no session evidence
> (screenshots, command output from a specific machine, room-specific answers) is included. Command
> examples below are generic reference examples, not captures from a completed session.

## Overview

Part 3 is the final room in TryHackMe's Linux Fundamentals module, moving from single commands and
permissions (Parts 1–2) to the day-to-day utilities and system-management concepts a working Linux user
relies on: text-processing tools, package management, task scheduling with cron, and where the system
keeps its logs. This guide covers each of those areas from public reference documentation, with generic
illustrative examples rather than room-specific tasks or answers.

## Core Concepts

### Text processing utilities

Beyond the basic `cat`/`grep` introduced earlier, Part 3-level material typically covers combining
utilities to search, transform, and summarize text — a skill set used constantly in log analysis:

| Command | Purpose |
|---|---|
| `grep -E 'pattern'` | Search with extended regular expressions |
| `cut -d',' -f2` | Extract a field by delimiter |
| `sort` | Sort lines (`-n` numeric, `-r` reverse, `-u` unique) |
| `uniq -c` | Collapse adjacent duplicate lines, counting occurrences |
| `sed 's/old/new/g'` | Stream-edit text, e.g. substitute text |
| `awk '{print $1}'` | Field-oriented text processing |
| `wc -l` | Count lines |

```text
$ cat access.log | awk '{print $1}' | sort | uniq -c | sort -rn | head -5
    182 203.0.113.10
     97 198.51.100.4
     ...
```

This pipeline extracts the first field (commonly the client IP in a web server log), sorts, counts
unique occurrences, and shows the top 5 most frequent source IPs — a canonical log-triage pattern.

### Package management

Debian/Ubuntu-family distributions use `dpkg` as the low-level package tool and `apt`/`apt-get` as the
higher-level frontend that resolves dependencies and fetches packages from configured repositories
(`/etc/apt/sources.list`):

```text
$ sudo apt update                 # refresh the package index
$ sudo apt upgrade                # upgrade installed packages
$ sudo apt install nmap           # install a package
$ sudo apt remove nmap            # remove a package
$ apt list --installed | grep nmap
```

Other distribution families use their own managers with equivalent roles: `dnf`/`yum` (Fedora/RHEL),
`pacman` (Arch), `zypper` (openSUSE). All follow the same conceptual flow: refresh a package index from
a trusted repository, resolve dependencies, then install/upgrade/remove.

### Task scheduling with cron

`cron` is a background daemon that runs commands on a fixed schedule defined in a **crontab**. Each
user can have their own crontab, edited with `crontab -e`, plus system-wide jobs in `/etc/crontab` and
`/etc/cron.d/`. A crontab line has five time fields followed by the command:

```text
# minute hour day-of-month month day-of-week command
  */15    *    *           *     *            /usr/local/bin/backup.sh
  0       2    *           *     0            /usr/local/bin/weekly-report.sh
```

`*/15 * * * *` means "every 15 minutes"; `0 2 * * 0` means "02:00 every Sunday." Listing a user's
current jobs is done with `crontab -l`.

```text
$ crontab -l
$ crontab -e
```

### System and application logging

Linux keeps most logs under `/var/log`. Common examples include `/var/log/auth.log` (or
`/var/log/secure` on RHEL-family systems) for authentication events, `/var/log/syslog` for general
system messages, and per-service subdirectories (e.g. `/var/log/nginx/`) for application logs. Modern
distributions using `systemd` also expose a unified structured log via `journalctl`:

```text
$ tail -n 20 /var/log/syslog
$ journalctl -u ssh --since "1 hour ago"
$ journalctl -p err -b
```

`journalctl -u <unit>` filters to one systemd service; `-p err` filters by priority; `-b` restricts to
the current boot.

### Process management and services

`ps`, `top`/`htop`, and `kill` are the standard tools for inspecting and controlling running processes:

```text
$ ps aux | grep nginx
$ top
$ kill -15 4821      # SIGTERM: ask the process to terminate gracefully
$ kill -9 4821       # SIGKILL: force-terminate immediately
```

Most modern distributions manage long-running services with `systemd`, controlled via `systemctl`:

```text
$ sudo systemctl status ssh
$ sudo systemctl restart ssh
$ sudo systemctl enable ssh    # start automatically on boot
```

### Environment variables and shell configuration files

Part 3-level material typically ties earlier navigation/file skills into configuring the shell
environment itself — `export`ing variables and understanding which startup file (`~/.bashrc`,
`~/.bash_profile`, `/etc/environment`) applies in which context. A fuller treatment of this, along with
scripting constructs, is in the [Linux Shells](../../easy/linux-shells/README.md) guide.

### Archiving as part of automation

Backup and log-rotation automation commonly combines the scheduling and text-processing pieces above
with archiving tools:

```text
$ tar -czf "/backups/logs-$(date +%F).tar.gz" /var/log/myapp/
$ find /backups -name "*.tar.gz" -mtime +30 -delete   # prune backups older than 30 days
```

This pattern — compress, timestamp, then prune anything past a retention window — is the basis of most
simple custom backup and log-rotation cron jobs (as distinct from dedicated tools like `logrotate`,
which ships with most distributions specifically to manage `/var/log` growth).

## Why It Matters for Security

- **Text-processing pipelines are the core of log analysis**, whether triaging a suspected breach,
  hunting for a specific IOC across gigabytes of logs, or building detection rules — `grep`/`awk`/`sed`
  fluency scales far better than manual review.
- **Package management is both an attack surface and a defense mechanism.** Outdated packages are a
  primary source of exploitable vulnerabilities, while unauthorized or spoofed repositories are a
  supply-chain risk; keeping `apt update && apt upgrade` current is baseline hygiene.
- **Cron jobs are a classic persistence and privilege-escalation vector.** An attacker who can write to
  a script that root's crontab executes gains root the next time it fires; conversely, defenders should
  audit `/etc/cron.d/`, `/etc/crontab`, and per-user crontabs as part of persistence hunting.
- **Centralized, immutable logging is foundational to incident response.** Without logs in `/var/log`
  or `journalctl`, there is no record to investigate after a compromise — which is why attackers
  frequently attempt log tampering as an anti-forensics step.

## Common Pitfalls / Misconfigurations

- **World-writable scripts referenced by a root cron job** — one of the most common realistic
  privilege-escalation paths: any user can edit the script, and root executes it on the next scheduled
  run.
- **Adding untrusted third-party APT repositories** or installing packages from unverified `.deb`
  files, bypassing the distribution's normal trust chain.
- **Never rotating or reviewing logs**, letting `/var/log` grow unbounded (a disk-exhaustion
  availability risk) while also never actually looking at the data for signs of compromise.
- **Overly broad or undocumented cron entries** that make it hard to distinguish legitimate
  automation from an attacker's persistence mechanism during an audit.

## Related TryHackMe Rooms in This Series

1. [Operating Systems: Introduction](../../easy/operating-systems-introduction/README.md)
2. [Operating System Security](../../easy/operating-system-security/README.md)
3. [Linux CLI Basics](../../easy/linux-cli-basics/README.md)
4. [Linux Fundamentals Part 1](../linux-fundamentals-part-1/README.md)
5. [Linux Fundamentals Part 2](../linux-fundamentals-part-2/README.md)
6. Linux Fundamentals Part 3 *(this room)*
7. [Linux Shells](../../easy/linux-shells/README.md)

## References

- [GNU grep manual](https://www.gnu.org/software/grep/manual/grep.html)
- [GNU sed manual](https://www.gnu.org/software/sed/manual/sed.html)
- [GNU awk (gawk) manual](https://www.gnu.org/software/gawk/manual/gawk.html)
- [crontab(5) — Linux man page](https://man7.org/linux/man-pages/man5/crontab.5.html)
- [cron(8) — Linux man page](https://man7.org/linux/man-pages/man8/cron.8.html)
- [Debian Administrator's Handbook — APT](https://debian-handbook.info/browse/stable/sect.apt-get.html)
- [journalctl(1) — systemd man page](https://www.freedesktop.org/software/systemd/man/journalctl.html)
- [Filesystem Hierarchy Standard (FHS) 3.0](https://refspecs.linuxfoundation.org/FHS_3.0/fhs-3.0.html)
- [ps(1) — Linux man page](https://man7.org/linux/man-pages/man1/ps.1.html)
- [kill(1) — Linux man page](https://man7.org/linux/man-pages/man1/kill.1.html)
- [systemctl(1) — systemd man page](https://www.freedesktop.org/software/systemd/man/systemctl.html)
- [logrotate(8) — Linux man page](https://man7.org/linux/man-pages/man8/logrotate.8.html)
