<!--
Fill this in from memory / your terminal history. Rough bullets are fine — I turn this into full
prose. The more specific you are (actual commands, actual output, actual wrong turns), the better
and more defensible the final article. Delete any section that doesn't apply. Don't worry about
grammar or completeness; I'll ask follow-up questions for anything that's too thin to write from.

Drop any screenshots you have into this same folder (_incoming/dns-in-detail/) — filenames
don't matter, I'll rename them. No screenshots is fine too.
-->

# DNS in Detail — raw notes

## Target / environment
- Domain(s) you looked up:
- Attack box / machine used (TryHackMe AttackBox / own Kali / other):

## Lookups performed
- Tools used (dig, nslookup, host, etc.) and the actual command(s) you ran:
- What DNS record types did you look up (A, AAAA, CNAME, NS, MX, TXT, SOA, PTR, SRV)?
- What the output actually showed (resolved IPs, TTLs, name servers, etc.):

## Resolution flow
- Did the room have you trace the stub → recursive → root → TLD → authoritative flow (e.g. with
  `dig +trace`)? What did that output look like?
- Anything that stood out (unexpected NS delegation, a CNAME chain, a low/high TTL):

## Wrong turns
- Anything you tried that didn't pan out, or a record/flag that confused you at first:

## Flags / answers
- Any flags you captured (I'll redact the actual values in the published article — just note
  where/how you got each one):
- Any other task answers worth remembering:

## What took longest / what you'd do differently
-

## Anything else worth noting
-
