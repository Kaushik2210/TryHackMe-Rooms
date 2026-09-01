# Linux Shells

**Platform:** TryHackMe · **Type:** Concept Guide (no personal run captured — see note below)
**Primary domain:** Linux / Operating Systems

> **Note:** This is a concept guide covering the material this TryHackMe room teaches, written from
> public documentation and reference material. It is not a personal walkthrough — no session evidence
> (screenshots, command output from a specific machine, room-specific answers) is included. Command
> examples below are generic reference examples, not captures from a completed session.

## Overview

A shell is the program that interprets the commands typed at a terminal (or received over SSH) and
turns them into system calls, process launches, and output. It is not part of the kernel — it runs as
an ordinary userland process — but it is the interface almost everyone uses to actually operate a Linux
system. This guide surveys the major Unix shells, the difference between an interactive session and a
shell script, core scripting constructs (variables, conditionals, loops), and environment configuration,
tying together the command-line skills covered elsewhere in this series into something reusable and
automatable.

## Core Concepts

### What a shell is, and the major families

| Shell | Notes |
|---|---|
| `sh` (Bourne shell / POSIX shell) | The original Unix shell specification; scripts written for `sh` are the most portable. |
| `bash` (Bourne Again SHell) | The GNU replacement for `sh`, and the default interactive/login shell on most Linux distributions. Superset of POSIX `sh` with extra features (arrays, `[[ ]]`, brace expansion). |
| `zsh` (Z shell) | POSIX-compatible with extensive interactive features (better completion, globbing); the default shell on modern macOS. |
| `dash` | A minimal, fast POSIX-compliant shell; on Debian/Ubuntu it's what `/bin/sh` actually points to, which is why `bash`-only syntax in a `#!/bin/sh` script can silently fail. |
| `fish` | A user-friendly interactive shell with a different (non-POSIX) scripting syntax, focused on out-of-the-box usability. |

A process's login shell is recorded in `/etc/passwd` (the last field per user); `chsh` changes it.
`echo $SHELL` or `echo $0` inside a script identifies which shell is currently interpreting commands.

### Interactive shell vs. shell script

Used interactively, a shell reads one command at a time from the terminal, executes it, and prints a
new prompt. A **shell script** is the same command language saved in a file and executed as a batch:

```bash
#!/bin/bash
# backup.sh - simple backup script
SRC="/home/alice/data"
DEST="/backups/data-$(date +%F).tar.gz"

tar -czf "$DEST" "$SRC"
echo "Backup written to $DEST"
```

The `#!/bin/bash` **shebang** on line 1 tells the kernel which interpreter to run the file with when
executed directly (`./backup.sh`, after `chmod +x backup.sh`). Without execute permission or a correct
shebang, the script must instead be invoked explicitly: `bash backup.sh`.

### Variables and command substitution

```bash
NAME="alice"
echo "Hello, $NAME"

FILES=$(ls /tmp)          # command substitution: capture a command's output
COUNT=$(ls /tmp | wc -l)
echo "There are $COUNT files in /tmp"
```

Variables are untyped strings by default; no `$` is used when *assigning*, but `$NAME` (or `${NAME}`
for clarity/concatenation) is required when *reading*. Quoting matters: `"$VAR"` preserves whitespace
and prevents word-splitting/globbing, while unquoted `$VAR` does not.

### Conditionals and loops

```bash
if [ -f "$DEST" ]; then
    echo "Backup exists"
elif [ -d "$DEST" ]; then
    echo "Unexpected: DEST is a directory"
else
    echo "No backup found"
fi

for f in /var/log/*.log; do
    echo "Processing $f"
done

while read -r line; do
    echo "Line: $line"
done < input.txt
```

`[ -f path ]` tests for a regular file, `[ -d path ]` for a directory, `[ -x path ]` for
executability — standard POSIX test operators also usable via the `test` command.

### Environment variables and shell configuration

Environment variables are inherited by every child process a shell spawns, distinguishing them from
ordinary shell variables that stay local:

```bash
export PATH="$HOME/bin:$PATH"
export EDITOR="vim"
```

`PATH` is the most operationally important one: an ordered, colon-separated list of directories the
shell searches, in order, when resolving a bare command name. Shells read configuration files on
startup to set these up — for `bash`, typically `/etc/profile` and `~/.bash_profile` for login shells,
and `~/.bashrc` for interactive non-login shells; `zsh` uses `~/.zshrc` analogously.

### Shell history

Interactive shells keep a record of previously run commands (`~/.bash_history` for `bash`), searchable
with `Ctrl+R` or reviewed with `history`. This is useful for productivity but also a forensic artifact
worth being aware of.

### Functions and script arguments

Scripts accept positional arguments and can define reusable functions, both frequently used together:

```bash
#!/bin/bash
log() {
    echo "[$(date +%T)] $1"
}

TARGET="$1"           # first argument passed to the script
if [ -z "$TARGET" ]; then
    echo "Usage: $0 <target>"
    exit 1
fi

log "Starting scan of $TARGET"
```

`$0` is the script's own name, `$1`..`$9` are positional arguments, `$@` expands to all arguments, and
`$#` gives the argument count. `exit <n>` sets the script's own exit status for the calling shell to
inspect via `$?`.

### Reverse and bind shells (concept only)

Because a shell is just a program that reads commands from stdin and writes to stdout/stderr, those
streams do not have to be a local terminal — they can be redirected over a network socket. This is the
conceptual basis of a **reverse shell** (the target connects outward to an attacker-controlled listener
and hands it a shell) versus a **bind shell** (the target listens on a port and hands a shell to
whoever connects in). Understanding that a shell is "just stdin/stdout/stderr redirected somewhere" is
the key insight — the mechanics of building and using these are covered in dedicated exploitation and
post-exploitation rooms rather than here, and no exploitation content or transferable payloads are
included in this concept guide.

### Job control

A shell can run commands in the background and manage multiple concurrent jobs within one session:

```text
$ long-running-task &        # run in the background, note the job number
[1] 4821
$ jobs                       # list background jobs
$ fg %1                      # bring job 1 to the foreground
$ bg %1                      # resume a stopped job in the background
```

`Ctrl+Z` suspends the current foreground job (`SIGTSTP`), after which `bg`/`fg` can resume it in the
background or foreground respectively.

## Why It Matters for Security

- **Scripting turns manual technique into repeatable tooling** — enumeration scripts, automated
  exploitation chains, and monitoring/alerting jobs are almost always shell scripts (or call out to
  them) under the hood.
- **`PATH` manipulation is a real attack technique.** If a privileged script or cron job calls a
  command by bare name without an absolute path, and an attacker can influence `PATH` or write to an
  earlier directory in it, they can hijack execution — a classic Linux privilege-escalation vector.
- **Shell history is both an OPSEC concern and a forensic goldmine.** `~/.bash_history` can leak
  credentials typed on the command line, and its absence/tampering can itself be a sign of anti-forensic
  activity during incident response.
- **`sh` vs `bash` behavioral differences cause real bugs**, and in security-sensitive automation
  (deployment scripts, CI pipelines) an assumption about which interpreter is running can lead to
  silent logic errors.

## Common Pitfalls / Misconfigurations

- **Unquoted variables in scripts**, which can lead to word-splitting or globbing on unexpected input —
  a frequent source of both bugs and, when handling untrusted data, command-injection-adjacent issues.
- **Running scripts as root without validating input** — a script that shells out with unsanitized
  user-controlled data (`eval`, unquoted substitution into a command) is a direct path to command
  injection.
- **Relying on relative paths or a mutable `PATH` inside privileged scripts/cron jobs.**
- **Assuming `#!/bin/sh` behaves like bash** on distributions where `/bin/sh` is actually `dash` or
  another minimal shell, causing bash-only syntax (`[[ ]]`, arrays) to fail or misbehave.
- **Leaving sensitive commands (passwords, tokens on the command line) in shell history**, where they
  persist in plaintext in `~/.bash_history`.

## Related TryHackMe Rooms in This Series

1. [Operating Systems: Introduction](../operating-systems-introduction/README.md)
2. [Operating System Security](../operating-system-security/README.md)
3. [Linux CLI Basics](../linux-cli-basics/README.md)
4. [Linux Fundamentals Part 1](../../fundamentals/linux-fundamentals-part-1/README.md)
5. [Linux Fundamentals Part 2](../../fundamentals/linux-fundamentals-part-2/README.md)
6. [Linux Fundamentals Part 3](../../fundamentals/linux-fundamentals-part-3/README.md)
7. Linux Shells *(this room)*

## References

- [GNU Bash Reference Manual](https://www.gnu.org/software/bash/manual/bash.html)
- [POSIX Shell & Utilities specification](https://pubs.opengroup.org/onlinepubs/9699919799/utilities/sh.html)
- [Dash — Debian package description](https://manpages.debian.org/stable/dash/dash.1.en.html)
- [Zsh documentation](https://zsh.sourceforge.io/Doc/)
- [Advanced Bash-Scripting Guide (TLDP)](https://tldp.org/LDP/abs/html/)
- [test(1) / `[` — Linux man page](https://man7.org/linux/man-pages/man1/test.1.html)
- [Bash Reference Manual — Job Control](https://www.gnu.org/software/bash/manual/html_node/Job-Control.html)
- [Bash Reference Manual — Shell Parameters (positional args)](https://www.gnu.org/software/bash/manual/html_node/Shell-Parameters.html)
- [signal(7) — Linux man page](https://man7.org/linux/man-pages/man7/signal.7.html)
