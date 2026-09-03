<!--
Fill this in from memory / your terminal history. Rough bullets are fine — I turn this into full
prose. The more specific you are (actual commands, actual output, actual wrong turns), the better
and more defensible the final article. Delete any section that doesn't apply. Don't worry about
grammar or completeness; I'll ask follow-up questions for anything that's too thin to write from.

Drop any screenshots you have into this same folder (_incoming/linux-shells/) — filenames don't
matter, I'll rename them. No screenshots is fine too.
-->

# Linux Shells — raw notes

## Target / environment
- Terminal/shell used (bash, zsh, other), and how you confirmed which one (`echo $SHELL`, `/etc/passwd`):

## Interactive shell vs. script
- Did you write an actual shell script? What did it do, and what was the shebang line?
- Any permission issues getting it to run (`chmod +x`, running via `bash script.sh` instead)?

## Variables and command substitution
- Any real variables you set and used, including command substitution (`$(...)`)? Actual lines:

## Conditionals and loops
- Did you write an `if`/`for`/`while` in a real script? What was it testing/iterating over?

## Environment variables and config files
- Did you `export` anything, or edit `~/.bashrc`/`~/.bash_profile`? What and why?
- Any `PATH` manipulation?

## Functions and script arguments
- Did your script use positional arguments (`$1`, `$@`, `$#`) or define a function? What was it for?

## Reverse/bind shells (concept)
- Did the room walk through the concept (not necessarily hands-on payloads) of shells over a network socket? What was your takeaway?

## Job control
- Any backgrounding (`&`), `jobs`, `fg`/`bg`, or `Ctrl+Z` you actually used?

## Task answers
- Any specific answers you had to submit (I'll redact flag-style values, but conceptual answers can stay visible in the final article):

## What took longest / what you'd do differently
-

## Anything else worth noting
-
