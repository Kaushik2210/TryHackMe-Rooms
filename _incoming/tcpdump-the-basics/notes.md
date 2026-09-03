<!--
Fill this in from memory / your terminal history. Rough bullets are fine — I turn this into full
prose. The more specific you are (actual commands, actual output, actual wrong turns), the better
and more defensible the final article. Delete any section that doesn't apply. Don't worry about
grammar or completeness; I'll ask follow-up questions for anything too thin to write from.

Drop any screenshots you have into this same folder (_incoming/tcpdump-the-basics/) — filenames
don't matter. No screenshots is fine too.
-->

# Tcpdump: The Basics — raw notes

## Target / environment
- Attack box / VM used, interface captured on:

## Basic invocation
- Actual `tcpdump` command(s) you ran (flags used: `-i`, `-n`/`-nn`, `-v`, `-c`, `-w`, `-r`, `-A`/`-X`, `-s`):
- What the output looked like:

## BPF filter expressions
- Actual filters you wrote (`host`, `src`/`dst`, `net`, `port`, `portrange`, boolean combos):
- What traffic they isolated:

## Reading TCP flags
- Any handshake/teardown traffic you read directly from the output (`S`, `S.`, `.`, `F`, `R`)?
- Any flag-bit filters you used (`tcp[tcpflags]`)?

## Writing/reading pcap files
- Did you write a capture with `-w` and later read it back or open it in Wireshark?

## Task answers
- Any specific answers or flags you had to find (I'll redact flag-style values):

## Where this connects to other rooms you've done
- e.g. Wireshark: The Basics, networking fundamentals rooms?

## What took longest / what you'd do differently
-

## Anything else worth noting
-
