# Completed Rooms

Source: user-provided screenshots of the TryHackMe "Completed rooms" profile page (2026-08-31).
URLs are not visible in the screenshots and are left blank — fill in each room's TryHackMe URL
before that room is processed, or confirm the standard pattern `https://tryhackme.com/room/<slug>`
is correct for all of them.

Difficulty is taken from the badge shown on each room card. Rooms with no difficulty badge (only an
"Info" link, no "Walkthrough") are marked `info` — these are typically non-graded/reading rooms and
may not warrant a full write-up; flag before processing.

| Room Name | URL | Difficulty | Date Completed | Path |
|---|---|---|---|---|
| How Websites Work | TBD | Easy | Unknown | Pre Security |
| Putting it all together | TBD | Easy | Unknown | Pre Security |
| DNS in Detail | TBD | Easy | Unknown | Pre Security |
| HTTP in Detail | TBD | Easy | Unknown | Pre Security |
| What is Networking? | TBD | Info | Unknown | Pre Security |
| Intro to LAN | TBD | Info | Unknown | Pre Security |
| OSI Model | TBD | Info | Unknown | Pre Security |
| Packets & Frames | TBD | Info | Unknown | Pre Security |
| Extending Your Network | TBD | Info | Unknown | Pre Security |
| Defensive Security Intro | TBD | Easy | Unknown | Pre Security |
| Careers in Cyber | TBD | Info | Unknown | Pre Security |
| Offensive Security Intro | TBD | Easy | Unknown | Pre Security |
| Virtualisation Basics | TBD | Unknown | Unknown | Pre Security |
| Client-Server Basics | TBD | Unknown | Unknown | Pre Security |
| Inside a Computer System | TBD | Unknown | Unknown | Pre Security |
| Computer Types | TBD | Unknown | Unknown | Pre Security |
| Operating System Security | TBD | Easy | Unknown | Pre Security |
| Operating Systems: Introduction | TBD | Easy | Unknown | Pre Security |
| Linux CLI Basics | TBD | Easy | Unknown | Pre Security |
| Data Representation | TBD | Easy | Unknown | Pre Security |
| Data Encoding | TBD | Easy | Unknown | Pre Security |
| JavaScript: Simple Demo | TBD | Medium | Unknown | Pre Security |
| Python: Simple Demo | TBD | Easy | Unknown | Pre Security |
| Windows Basics | TBD | Easy | Unknown | Pre Security |
| Cloud Computing Fundamentals | TBD | Easy | Unknown | Pre Security |
| Windows CLI Basics | TBD | Easy | Unknown | Pre Security |
| The CIA Triad | TBD | Easy | Unknown | Pre Security |
| Database SQL Basics | TBD | Easy | Unknown | Pre Security |
| Cryptography Concepts | TBD | Unknown | Unknown | Pre Security |
| Become a Hacker | TBD | Unknown | Unknown | Pre Security |
| Become a Defender | TBD | Unknown | Unknown | Pre Security |
| Search Skills | TBD | Unknown | Unknown | Pre Security |
| Linux Fundamentals Part 1 | TBD | Info | Unknown | Complete Beginner |
| Linux Fundamentals Part 2 | TBD | Info | Unknown | Complete Beginner |
| Linux Fundamentals Part 3 | TBD | Info | Unknown | Complete Beginner |
| Windows Fundamentals 1 | TBD | Info | Unknown | Complete Beginner |
| Windows Fundamentals 2 | TBD | Info | Unknown | Complete Beginner |
| Windows Fundamentals 3 | TBD | Info | Unknown | Complete Beginner |
| Wireshark: The Basics | TBD | Easy | Unknown | Cyber Security 101 |
| Active Directory Basics | TBD | Easy | Unknown | Cyber Security 101 |
| Windows Command Line | TBD | Easy | Unknown | Cyber Security 101 |
| Networking Concepts | TBD | Easy | Unknown | Cyber Security 101 |
| Tcpdump: The Basics | TBD | Easy | Unknown | Cyber Security 101 |
| Networking Essentials | TBD | Easy | Unknown | Cyber Security 101 |
| Networking Core Protocols | TBD | Easy | Unknown | Cyber Security 101 |
| Networking Secure Protocols | TBD | Easy | Unknown | Cyber Security 101 |
| Windows PowerShell | TBD | Easy | Unknown | Cyber Security 101 |
| Linux Shells | TBD | Easy | Unknown | Cyber Security 101 |
| Junior Security Analyst Intro | TBD | Easy | Unknown | Cyber Security 101 |
| Guided Pentest: Web | TBD | Easy | Unknown | Cyber Security 101 |

## Open items before pipeline can run

1. **URLs missing** for every room — need the actual `tryhackme.com/room/...` links.
2. **Dates completed missing** — screenshots didn't show a date column; the "Yearly activity" tab
   on the profile may have this.
3. **Difficulty unknown** for: Virtualisation Basics, Client-Server Basics, Inside a Computer System,
   Computer Types, Cryptography Concepts, Become a Hacker, Become a Defender, Search Skills
   (screenshots were cropped before the badge).
4. **"Info"-only rooms** (no Walkthrough badge, e.g. OSI Model, Careers in Cyber, all three Linux/Windows
   Fundamentals rooms) are typically reading/theory rooms with no hands-on tasks — confirm whether these
   should get full write-ups or a shorter concept-only treatment.
5. **`_data/profile.md` does not exist yet** — needed before Phase 5 (Authoring) for tone/depth.
6. **No `_incoming/<room-slug>/notes.md` or screenshots exist for any room yet** — this is the
   pipeline's hard blocker per the master prompt's "never fabricate evidence" rule. Nothing can be
   authored until at least one room has real notes.
