# Python: Simple Demo

**Platform:** TryHackMe · **Type:** Concept Guide (no personal run captured — see note below)
**Primary domain:** Programming Fundamentals

> **Note:** This is a concept guide covering the material this TryHackMe room teaches, written from
> public documentation and reference material. It is not a personal walkthrough — no session evidence
> (screenshots, command output, room-specific answers) is included. Code examples below are generic
> reference examples, not captures from a completed session.

## Overview

Python is a high-level, interpreted, dynamically-typed language originally released by Guido van
Rossum in 1991 and now maintained by the Python Software Foundation. Its design philosophy favors
readability — significant whitespace instead of braces, a small set of orthogonal keywords, and a
"batteries included" standard library. Introductory rooms in this space typically cover variables,
data types, control flow, functions, and basic I/O, since these are the building blocks every later
script (automation, parsing, exploit PoCs) is built from. This guide covers the same ground with
generic, runnable examples.

## Core Concepts

### Variables and Dynamic Typing

Python variables are names bound to objects; the interpreter infers type at runtime rather than
requiring a declaration.

```python
name = "TryHackMe"      # str
room_count = 3            # int
difficulty = 4.5          # float
is_free = True             # bool

print(type(name), type(room_count), type(difficulty), type(is_free))
# <class 'str'> <class 'int'> <class 'float'> <class 'bool'>
```

`type()` returns the runtime class of an object. Because binding is dynamic, the same name can be
rebound to a different type later — Python will not stop you, which is convenient but also a common
source of bugs in larger scripts.

### Strings and f-strings

String formatting is central to almost every security script (building payloads, formatting output).

```python
target = "10.10.10.10"
port = 443
url = f"https://{target}:{port}/login"
print(url)
# https://10.10.10.10:443/login
```

f-strings (introduced in Python 3.6, PEP 498) evaluate the expression inside `{}` at runtime and
insert the result into the string — cleaner than the older `%` or `.format()` styles.

### Control Flow

```python
def classify_port(port: int) -> str:
    if port == 80 or port == 443:
        return "web"
    elif port == 22:
        return "ssh"
    elif port < 1024:
        return "well-known"
    else:
        return "high/ephemeral"

for p in (22, 80, 8443):
    print(p, "->", classify_port(p))
# 22 -> ssh
# 80 -> web
# 8443 -> high/ephemeral
```

`if`/`elif`/`else` chains and `for ... in` loops over iterables (lists, tuples, ranges, file handles)
are the two workhorses of control flow. Note the type hints (`port: int`, `-> str`) — optional in
Python but increasingly standard practice for readability and static analysis via tools like `mypy`.

### Functions, Lists, and Dictionaries

```python
def parse_ports(raw: str) -> list[int]:
    """Turn '22,80,443' into [22, 80, 443]."""
    return [int(p.strip()) for p in raw.split(",")]

ports = parse_ports("22, 80, 443")
services = {22: "ssh", 80: "http", 443: "https"}

for p in ports:
    print(p, services.get(p, "unknown"))
# 22 ssh
# 80 http
# 443 https
```

The list comprehension `[int(p.strip()) for p in raw.split(",")]` is idiomatic Python: it builds a
new list by applying an expression to each element of an iterable in one line. `dict.get(key,
default)` avoids a `KeyError` when a key may be absent — important when parsing untrusted input such
as scan results or user-supplied data.

### File I/O and Modules

```python
# reading a wordlist-style file safely
with open("targets.txt") as f:
    targets = [line.strip() for line in f if line.strip()]

import sys
if len(sys.argv) > 1:
    print("Argument received:", sys.argv[1])
```

The `with` statement is a context manager: it guarantees the file handle is closed even if an
exception occurs inside the block. `sys.argv` is the standard way small CLI scripts read
command-line arguments without pulling in a heavier library like `argparse`.

### Exceptions

```python
try:
    port = int(input("Port: "))
except ValueError:
    print("Not a valid integer")
else:
    print("Parsed port:", port)
finally:
    print("Done parsing")
```

`try`/`except` lets a script fail gracefully on malformed input instead of crashing — critical when a
script processes attacker-controlled or otherwise untrusted data.

## Why It Matters for Security

Python is the dominant scripting language across offensive and defensive security tooling. Frameworks
and tools such as `impacket`, `scapy`, much of `pwntools`, large parts of Metasploit's auxiliary
tooling ecosystem, and countless one-off exploit proof-of-concepts are written in Python because it is
fast to prototype in, has first-class libraries for networking (`socket`, `requests`), binary parsing
(`struct`), and process control (`subprocess`), and runs unmodified across Linux, macOS, and Windows.
Analysts use it to automate log parsing and IOC extraction; red teamers use it to write custom
payloads and C2 glue code; CTF players use it to script brute-forcers and format-string exploits.
Fluency in the basics shown above — string formatting, list/dict manipulation, file I/O, and
exception handling — is a prerequisite for almost every intermediate security-automation room that
follows an intro-to-Python room in a learning path.

## Common Pitfalls / Misconfigurations

- **Mixing Python 2 and 3 idioms.** Python 2 reached end-of-life in January 2020; `print` as a
  statement, integer-division-by-default, and implicit str/bytes coercion are all Python 2 behaviors
  that will break or silently misbehave under Python 3.
- **Not pinning or isolating dependencies.** Installing packages globally with `pip install` can
  clash with system packages; the standard fix is a virtual environment (`python -m venv .venv`).
- **Using `eval()`/`exec()` on untrusted input.** These execute arbitrary Python and are a common
  self-inflicted code-execution vulnerability in student scripts and even production tooling.
- **String-formatting SQL or shell commands instead of using parameterized APIs** (`subprocess.run(
  [...], shell=False)` over `os.system()`), which mirrors the SQL injection problem covered in
  database-focused rooms.
- **Off-by-one errors with `range()`** — `range(n)` produces `0..n-1`, not `1..n`, a frequent
  beginner mistake when iterating over ports or indices.
- **Catching bare `except:`** — swallows every exception including `KeyboardInterrupt` and
  `SystemExit`, hiding real bugs; catch specific exception types instead.

## Related TryHackMe Rooms in This Series

This room typically sits early in a learning path alongside other "fundamentals" rooms — see
`../database-sql-basics/README.md` for the corresponding database-scripting fundamentals, and
`../cloud-computing-fundamentals/README.md` for infrastructure concepts that later Python automation
rooms (e.g. interacting with cloud provider APIs) build on.

## References

- Python 3 official tutorial: https://docs.python.org/3/tutorial/index.html
- Python `venv` documentation: https://docs.python.org/3/library/venv.html
- PEP 498 — Literal String Interpolation (f-strings): https://peps.python.org/pep-0498/
- Python `subprocess` module documentation: https://docs.python.org/3/library/subprocess.html
- Python built-in exceptions reference: https://docs.python.org/3/library/exceptions.html
- Python Software Foundation, "Sunsetting Python 2": https://www.python.org/doc/sunset-python-2/
