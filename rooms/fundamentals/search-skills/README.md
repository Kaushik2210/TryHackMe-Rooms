# Search Skills

**Platform:** TryHackMe · **Type:** Concept Guide (no personal run captured — see note below)
**Primary domain:** Security Fundamentals

> **Note:** This is a concept guide covering the material this TryHackMe room teaches, written from
> public documentation and reference material. It is not a personal walkthrough — no session evidence
> (screenshots, command output, room-specific answers) is included.

## Overview

Effective searching is a foundational security skill that sits underneath almost everything else a
practitioner does: researching an unfamiliar error message, finding a CVE's technical writeup,
scoping an organization's internet-facing footprint during reconnaissance, or locating vendor
documentation for an obscure protocol. This guide covers advanced search operators (commonly grouped
under the label "Google dorking"), how to structure a research query so it converges quickly, and how
these techniques map onto the reconnaissance phase of both offensive engagements and OSINT
investigations. None of this requires special tools — it is a discipline built entirely on search
engines and public documentation, which is exactly why it is taught early and revisited constantly.

## Core Concepts

### Advanced search operators

Modern search engines (Google, Bing, DuckDuckGo) support a syntax of operators that let a query target
specific fields of a page rather than just its body text. The operators that show up most often in a
security context are:

- **`site:`** — restricts results to a single domain or subdomain, e.g. `site:example.com` or
  `site:*.example.com` to include subdomains. This is the anchor operator for scoping reconnaissance
  to a single target rather than the open web.
- **`filetype:`** (or `ext:`) — restricts results to a specific file extension, e.g.
  `filetype:pdf`, `filetype:xlsx`, `filetype:conf`. Useful for finding exposed configuration files,
  internal documents, or presentation decks that were unintentionally indexed.
- **`intitle:`** — matches text that appears in the HTML `<title>` of a page, e.g.
  `intitle:"index of"` — a classic dork for finding open directory listings.
- **`inurl:`** — matches text that appears in the URL path itself, e.g. `inurl:admin` or
  `inurl:login`, useful for surfacing administrative or authentication endpoints.
- **`intext:`** — matches text anywhere in the page body, useful for narrowing to pages that mention
  a specific string (an error message, a software version banner, an internal hostname).
- **Boolean and grouping operators** — quotation marks for exact phrases (`"internal use only"`),
  the minus sign to exclude terms (`vpn -cisco`), the pipe or `OR` for alternation, and parentheses to
  group clauses. Combining operators narrows a broad query into something precise, e.g.
  `site:example.com filetype:pdf intext:"confidential"`.

These same operators generalize to other search surfaces used in reconnaissance and OSINT: GitHub code
search for leaked secrets or internal repo references, LinkedIn's people-search filters for employee
enumeration, and specialized engines like Shodan or Censys (`org:`, `port:`, `product:` filters) for
internet-facing device discovery.

### The Google Hacking Database (GHDB)

The GHDB, maintained on Exploit-DB, is a community-curated archive of dork queries organized by
category (footholds, files containing juicy info, vulnerable files, error messages that disclose paths,
etc.). It formalizes what "Google hacking" means in practice: search operators were never a
vulnerability in themselves, but a way of finding content that was already public yet never meant to be
discoverable — a misconfigured web server that allowed directory listing, a backup file left in a web
root, or a spreadsheet an employee uploaded without realizing it was crawlable. Reviewing GHDB
categories is a fast way to understand the taxonomy of things dorking commonly surfaces, without needing
to run queries against a real target.

### Search as part of the reconnaissance phase

In both formal penetration testing methodology and cyber threat modeling, information gathering precedes
any active interaction with a target. MITRE ATT&CK models this explicitly as the **Reconnaissance**
tactic (TA0043), which includes techniques such as *Search Open Websites/Domains* (T1593), *Search
Open Technical Databases* (T1596), and *Search Victim-Owned Websites* (T1594) — all of which describe
adversaries using exactly the search techniques above to build a picture of a target before attempting
access. The same techniques, applied by defenders against their own organization, are called **attack
surface discovery** or **passive reconnaissance for defensive purposes**: finding what an attacker
would find, before they find it. This is why dorking shows up on both sides of the fence — the
technique is identical, only the intent and authorization differ.

### Structuring a research query for technical problem-solving

Search skills apply just as much to day-to-day technical troubleshooting as they do to reconnaissance.
A few habits noticeably improve convergence speed:

- **Search the exact error text in quotes**, stripped of anything environment-specific (file paths,
  usernames, timestamps), so the engine matches other reports of the identical problem.
- **Add the specific software/version/vendor name** rather than searching generically — "nginx 1.25
  worker_connections exceeded" converges faster than "server too many connections."
- **Prefer primary sources** — vendor documentation, RFCs, CVE/NVD entries, MITRE ATT&CK technique
  pages — over aggregator blogs, which frequently paraphrase (and sometimes garble) the original.
- **Use site-scoped search on documentation domains** when a vendor's own search is weak, e.g.
  `site:learn.microsoft.com kerberos delegation`.
- **Iterate the query rather than the source** — if the first few results are off-target, adjust
  operators and terms before giving up and switching to a completely different search engine.

## Why It Matters for Security

Search proficiency is a force multiplier across every security role. A SOC analyst triaging an alert
searches for the exact process name or registry key to determine if it is a known false positive or a
documented technique. A penetration tester's reconnaissance phase is largely search-driven before a
single packet touches the target — company structure, technology stack, exposed subdomains, and public
employee information are frequently obtainable through search alone. A threat intelligence analyst
tracking a campaign searches for infrastructure overlap, reused code, or public reporting from other
researchers. Even incident response benefits directly: searching a suspicious hash, domain, or IOC
against public sources is often the fastest path to attribution or scoping. Because dorking surfaces
real, sometimes sensitive exposures, security teams also use it defensively — running GHDB-style
queries against their own domains periodically to catch accidental exposure (an open directory listing,
an indexed internal spreadsheet, a forgotten `.git` folder) before an outside party does.

## Common Mistakes

- **Treating dorking as inherently authorized.** Running reconnaissance-style search queries against a
  target you do not have explicit permission to test can cross into unauthorized activity depending on
  what is retrieved and how it is used — dorking against your own assets or an assigned lab target is
  fine; running the same queries against a third party without authorization is not.
- **Over-broad queries that return noise.** Skipping the narrowing operators (`site:`, `filetype:`,
  exact phrases) leads to thousands of irrelevant results and wastes time; specificity is the entire
  point of the technique.
- **Ignoring that search engines index selectively and lag reality.** A dork may surface a page that
  was already fixed, or miss content that was never crawled — search results are a snapshot, not a live
  view of a target's current state.
- **Relying on a single engine.** Google, Bing, and DuckDuckGo index differently and support slightly
  different operator sets; a query that returns nothing on one engine can return real hits on another.
- **Skipping primary sources during technical research.** Pasting an error into a forum search and
  taking the first answer at face value, without cross-checking against vendor documentation or a CVE
  record, is a common source of misdiagnosis.

## Related TryHackMe Rooms in This Series

- [Offensive Security Intro](../../easy/offensive-security-intro/README.md) — search skills feed
  directly into the reconnaissance phase described there.
- [Defensive Security Intro](../../easy/defensive-security-intro/README.md) — the same search
  techniques, applied against your own organization, support proactive attack-surface discovery.
- [Careers in Cyber](../careers-in-cyber/README.md)
- [Become a Hacker](../become-a-hacker/README.md)
- [The CIA Triad](../../easy/the-cia-triad/README.md)

## References

- MITRE ATT&CK, *Reconnaissance (TA0043)*: https://attack.mitre.org/tactics/TA0043/
- MITRE ATT&CK, *Search Open Websites/Domains (T1593)*: https://attack.mitre.org/techniques/T1593/
- MITRE ATT&CK, *Search Open Technical Databases (T1596)*: https://attack.mitre.org/techniques/T1596/
- MITRE ATT&CK, *Search Victim-Owned Websites (T1594)*: https://attack.mitre.org/techniques/T1594/
- Exploit-DB, *Google Hacking Database (GHDB)*: https://www.exploit-db.com/google-hacking-database
- OWASP, *Web Security Testing Guide — Information Gathering*: https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/01-Information_Gathering/
- Google, *Refine web searches (search operators reference)*: https://support.google.com/websearch/answer/2466433
- NIST SP 800-115, *Technical Guide to Information Security Testing and Assessment*: https://csrc.nist.gov/pubs/sp/800/115/final
