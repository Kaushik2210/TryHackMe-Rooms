<!--
Fill this in from memory / your terminal history. Rough bullets are fine — I turn this into full
prose. The more specific you are (actual commands, actual output, actual wrong turns), the better
and more defensible the final article. Delete any section that doesn't apply. Don't worry about
grammar or completeness; I'll ask follow-up questions for anything that's too thin to write from.

Drop any screenshots you have into this same folder (_incoming/operating-system-security/) —
filenames don't matter, I'll rename them. No screenshots is fine too.
-->

# Operating System Security — raw notes

## Target / environment
- Target IP or hostname / deployed machine:
- Attack box used (TryHackMe AttackBox / own Kali / other):

## Permissions and ownership work
- What files/directories did you inspect with `ls -l`, and what did the permission strings show?
- Any `chmod`/`chown`/`chgrp` you actually ran — commands and results:
- Did the room have you find or exploit SUID/SGID binaries? What commands (`find / -perm -4000`, etc.) and what did you find?

## SSH / remote access work
- Did you connect via SSH, generate a keypair, or edit `sshd_config`? Actual commands:
- Any key-permission issues you hit (`chmod 600`, refused keys, etc.):
- What did `authorized_keys` / `sudoers` look like by the end?

## sudo / privilege model
- Any `sudo`, `visudo`, or sudoers entries you looked at or edited:
- Anything about least-privilege or privilege separation the room made you demonstrate hands-on:

## ACLs, capabilities, or MAC (SELinux/AppArmor) — if the room touched these
- What did `getfacl`/`setfacl` or `getcap`/`setcap` commands actually show/do:
- Any SELinux/AppArmor mode-checking or policy work:

## Task answers
- Any specific answers you had to submit (I'll redact flag-style values, but conceptual answers can stay visible in the final article):

## What took longest / what you'd do differently
-

## Anything else worth noting
-
