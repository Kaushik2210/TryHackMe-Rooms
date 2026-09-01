# Operating System Security

**Platform:** TryHackMe · **Type:** Concept Guide (no personal run captured — see note below)
**Primary domain:** Linux / Operating Systems

> **Note:** This is a concept guide covering the material this TryHackMe room teaches, written from
> public documentation and reference material. It is not a personal walkthrough — no session evidence
> (screenshots, command output from a specific machine, room-specific answers) is included. Command
> examples below are generic reference examples, not captures from a completed session.

## Overview

Operating system security is the set of mechanisms an OS uses to keep users, processes, and data
separated from one another so that a compromise or mistake in one part of the system cannot freely
spread to the rest. On a multi-user system this boils down to a handful of enforced boundaries: who
owns a file, who can read/write/execute it, which user a process runs as, and what that user is allowed
to do once authenticated. This guide surveys the two pillars that TryHackMe's OS-security material
builds on — the Linux discretionary access control (permission) model and secure remote authentication
via SSH — and connects them to the privilege-escalation and hardening work covered later in the series.

## Core Concepts

### The Unix/Linux permission model

Every file and directory on a Linux system has an owning user, an owning group, and a permission
string visible via `ls -l`:

```text
$ ls -l /etc/shadow /usr/bin/passwd
-rw-r----- 1 root shadow    1156 Jan  3 10:02 /etc/shadow
-rwsr-xr-x 1 root root     68208 Jan  3 09:58 /usr/bin/passwd
```

The ten-character string decomposes as: file type (`-`, `d`, `l`, …), then three permission triads —
owner, group, others — each expressing **r**ead, **w**rite, **e**xecute:

| Symbolic | Octal | Meaning on a file | Meaning on a directory |
|---|---|---|---|
| `r` | 4 | View contents | List entries (`ls`) |
| `w` | 2 | Modify contents | Create/delete/rename entries |
| `x` | 1 | Execute as a program/script | Enter (`cd`) or traverse |

Octal notation sums the bits per triad: `rwxr-xr--` = `754` (owner 7 = rwx, group 5 = r-x, others 4 =
r--). Changing permissions and ownership uses `chmod` and `chown`/`chgrp`:

```text
$ chmod 640 notes.txt        # owner rw-, group r--, others ---
$ chmod u+x deploy.sh        # add execute for the owner only
$ chown alice:developers report.csv
```

Two special bits matter for security beyond the base rwx model:

- **SUID** (`chmod u+s`, shows as `s` in the owner execute slot) — the binary runs with the *file
  owner's* privileges rather than the caller's. `/usr/bin/passwd` is SUID root so any user can update
  `/etc/shadow`, which is otherwise root-only.
- **SGID** (`chmod g+s`) — on a binary, runs with the group's privileges; on a directory, new files
  inherit the directory's group instead of the creator's primary group.
- **Sticky bit** (`chmod +t`, seen on `/tmp`) — inside a sticky directory, users can only delete or
  rename files they own, even if the directory itself is world-writable.

### Users, groups, and privilege separation

Linux tracks accounts in `/etc/passwd` (username, UID, GID, home directory, login shell) and password
hashes in the root-only-readable `/etc/shadow`. Group membership lives in `/etc/group`. UID `0` is
always root/superuser regardless of the username; anything else is an unprivileged account whose access
is mediated entirely through the permission bits above (plus, on hardened systems, mechanisms like
`sudo`, POSIX capabilities, or SELinux/AppArmor policy).

`sudo` lets an authorized user run specific commands as another user (usually root) without sharing the
root password, governed by `/etc/sudoers` (edited safely with `visudo`, which validates syntax before
saving). The principle behind all of this is **least privilege**: every process and account should hold
only the access it needs for its job, nothing more.

### SSH: secure remote administration

Secure Shell (SSH) is the standard protocol for encrypted remote login and command execution, replacing
legacy cleartext protocols like Telnet and rsh. A connection negotiates a symmetric session key via
asymmetric key exchange, then authenticates the user by one of two common methods:

```text
$ ssh alice@203.0.113.10                 # password authentication
$ ssh -i ~/.ssh/id_ed25519 alice@203.0.113.10   # public-key authentication
```

**Key-based authentication** is preferred for security: the user holds a private key (never
transmitted), and the server holds the matching public key in `~/.ssh/authorized_keys`. Generating a
keypair and the resulting file permissions matter:

```text
$ ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519
$ chmod 700 ~/.ssh
$ chmod 600 ~/.ssh/id_ed25519 ~/.ssh/authorized_keys
```

SSH itself is configured server-side in `/etc/ssh/sshd_config`, where an administrator can disable
root login (`PermitRootLogin no`), disable password auth in favor of keys only
(`PasswordAuthentication no`), and change the listening port.

### Access control lists and capabilities

Traditional Unix permissions only support one owner and one group per file. Two extensions relax that
constraint:

- **POSIX ACLs** (`getfacl`/`setfacl`) let an administrator grant fine-grained access to additional
  named users or groups beyond the standard owner/group/other triad — useful when a file needs to be
  shared with one extra team without loosening permissions for everyone.
- **Linux capabilities** (`getcap`/`setcap`) split the traditionally all-or-nothing root privilege into
  discrete units, e.g. `CAP_NET_BIND_SERVICE` (bind to ports below 1024) or `CAP_SYS_ADMIN`. A binary
  can be granted just the capability it needs instead of full SUID-root, which is a meaningfully smaller
  attack surface — `ping` on modern distributions typically uses `cap_net_raw` instead of full setuid
  root, for example.

```text
$ getfacl shared-report.csv
$ setfacl -m u:bob:rw shared-report.csv
$ getcap /usr/bin/ping
/usr/bin/ping cap_net_raw=ep
```

### Mandatory access control: SELinux and AppArmor

Discretionary access control (the owner-set permission bits above) means the resource owner decides who
gets access. Many hardened distributions layer **mandatory access control (MAC)** on top, where a
system-wide policy — not the file owner — has final say over what a process can do, even as root:

- **SELinux** (Red Hat/Fedora/CentOS family) labels every process and file with a security context and
  enforces a policy matching those labels; a compromised web server process confined by an SELinux
  policy cannot read arbitrary files even if the Unix permission bits would technically allow it.
- **AppArmor** (Debian/Ubuntu/SUSE family) achieves a similar goal via per-application profiles keyed to
  file paths rather than labels, generally considered simpler to author than SELinux policy.

Both operate in an enforcing or a permissive/complaining mode; the latter logs would-be violations
without blocking them, useful while developing a new policy.

### Authentication beyond passwords

Modern Linux authentication is pluggable via **PAM** (Pluggable Authentication Modules), which lets an
administrator layer additional factors — one-time codes (`pam_google_authenticator`), account lockout
after failed attempts (`pam_faillock`), or password-complexity requirements — onto standard login, `su`,
and `sudo` flows without modifying the applications themselves. This is the mechanism behind
multi-factor SSH login and account-lockout policies referenced throughout hardened Linux baselines
(e.g. CIS Benchmarks).

## Why It Matters for Security

- **Permissions are the last line of defense.** Most local privilege-escalation techniques on Linux —
  abusing a misconfigured SUID binary, a world-writable script run by a cron job, or an overly
  permissive sudoers entry — are permission-model failures, not kernel exploits.
- **Least privilege limits blast radius.** A web server process running as an unprivileged `www-data`
  account that gets compromised cannot read `/etc/shadow` or modify system binaries the way a
  root-owned process could.
- **SSH is the primary remote-attack and remote-defense surface.** Brute-forceable password auth,
  reused keys, or a permissive `sshd_config` are among the most common initial-access vectors against
  Linux servers, which is why key-based auth and `fail2ban`-style rate limiting are baseline hardening.
- **Auditing SUID/SGID binaries and sudoers rules is routine both offensively and defensively** — the
  same `find / -perm -4000` command a penetration tester runs to hunt for privesc paths is also what a
  defender should run periodically to catch drift from an intended baseline.

## Common Pitfalls / Misconfigurations

- **Unnecessary SUID bits.** Custom scripts or third-party binaries marked SUID root (often for
  convenience) are one of the most common privilege-escalation vectors found in CTFs and real audits.
- **World-writable files owned by root**, or scripts executed by a cron job that are writable by an
  unprivileged user — either lets an attacker inject code that later runs with elevated privileges.
- **Overly broad sudoers entries**, e.g. granting `NOPASSWD: ALL` or access to an editor/interpreter
  that can spawn a shell (`vim`, `less`, `python`), which trivially escalates to full root.
- **Password-only SSH authentication** exposed to the internet, especially with the default port and
  no rate limiting, invites credential-stuffing and brute-force attacks.
- **Reused or improperly protected private keys** — an `id_rsa` with world-readable permissions
  (`ssh` will actually refuse to use an insecurely-permissioned key) or a key shared across many hosts
  turns one compromised machine into many.

## Related TryHackMe Rooms in This Series

1. [Operating Systems: Introduction](../operating-systems-introduction/README.md)
2. Operating System Security *(this room)*
3. [Linux CLI Basics](../linux-cli-basics/README.md)
4. [Linux Fundamentals Part 1](../../fundamentals/linux-fundamentals-part-1/README.md)
5. [Linux Fundamentals Part 2](../../fundamentals/linux-fundamentals-part-2/README.md)
6. [Linux Fundamentals Part 3](../../fundamentals/linux-fundamentals-part-3/README.md)
7. [Linux Shells](../linux-shells/README.md)

## References

- [chmod(1) — Linux man page](https://man7.org/linux/man-pages/man1/chmod.1.html)
- [chmod(2) — special permission bits (setuid, setgid, sticky)](https://man7.org/linux/man-pages/man2/chmod.2.html)
- [passwd(5) — Linux man page](https://man7.org/linux/man-pages/man5/passwd.5.html)
- [shadow(5) — Linux man page](https://man7.org/linux/man-pages/man5/shadow.5.html)
- [sudoers(5) — Linux man page](https://man7.org/linux/man-pages/man5/sudoers.5.html)
- [sshd_config(5) — OpenSSH server configuration](https://man.openbsd.org/sshd_config)
- [ssh-keygen(1) — OpenSSH key generation](https://man.openbsd.org/ssh-keygen.1)
- [OpenSSH project documentation](https://www.openssh.com/manual.html)
- [setfacl(1) / getfacl(1) — Linux man pages](https://man7.org/linux/man-pages/man1/setfacl.1.html)
- [capabilities(7) — Linux man page](https://man7.org/linux/man-pages/man7/capabilities.7.html)
- [SELinux Project documentation](https://selinuxproject.org/page/Main_Page)
- [AppArmor documentation (Ubuntu)](https://ubuntu.com/server/docs/apparmor)
- [Linux-PAM System Administrator's Guide](https://www.man7.org/linux/man-pages/man8/PAM.8.html)
