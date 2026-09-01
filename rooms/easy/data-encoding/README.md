# Data Encoding

**Platform:** TryHackMe · **Type:** Concept Guide (no personal run captured — see note below)
**Primary domain:** Computing Fundamentals

> **Note:** This is a concept guide covering the material this TryHackMe room teaches, written from
> public documentation and reference material. It is not a personal walkthrough — no session evidence
> (screenshots, command output, room-specific answers) is included.

## Overview

Encoding is the process of transforming data from one representation into another so it can be safely stored, transmitted, or displayed by a system with constraints on what characters or byte values it can handle. Base64, URL-encoding, and UTF-8 are three encodings that appear constantly in security work — in HTTP requests, in malware droppers, in log files, and in CTF challenges. This room-level topic covers how each of these works mechanically, with worked byte-level examples, and addresses directly the single most common misconception in this space: **encoding is not encryption**, and treating it as if it provides confidentiality is a real, exploitable mistake.

## Core Concepts

### Base64 (RFC 4648)

Base64 converts arbitrary binary data into a string built only from the 64 characters `A–Z`, `a–z`, `0–9`, `+`, and `/` (with `=` used as padding), as standardised in RFC 4648, "The Base16, Base32, and Base64 Data Encodings." It does this by taking the input 8 bits (1 byte) at a time, regrouping those bits into chunks of 6 bits, and mapping each 6-bit value (0–63) to one of the 64 output characters. Because 6 bits produces only 64 possible values, and a byte is 8 bits, encoded output is always about 33% larger than the original binary input.

Worked example — encoding the ASCII string `Hi!`:

```
Step 1: Get the ASCII value of each character, in binary (8 bits each)
  H = 72  = 01001000
  i = 105 = 01101001
  ! = 33  = 00100001

Step 2: Concatenate all the bits (24 bits total)
  010010000110100100100001

Step 3: Regroup into 6-bit chunks
  010010  000110  100100  100001

Step 4: Convert each 6-bit chunk to decimal, then map to the Base64 alphabet
  010010 = 18 → 'S'
  000110 =  6 → 'G'
  100100 = 36 → 'k'
  100001 = 33 → 'h'

Result: "Hi!" → "SGkh"
```

If the input length isn't a multiple of 3 bytes, `=` padding characters are appended to indicate how many bits of the final group are "real" — a single `=` means the last group encoded 2 bytes, `==` means it encoded only 1.

### URL encoding / percent-encoding (RFC 3986)

URLs can only safely contain a limited set of characters. **Percent-encoding**, defined by RFC 3986 ("Uniform Resource Identifier (URI): Generic Syntax"), represents any byte outside that safe set as a `%` followed by its two-digit hexadecimal value. Reserved characters with special meaning in a URL (`&`, `=`, `?`, `/`, `#`, and space) must be percent-encoded when they appear as literal data rather than as URL syntax.

Worked example — encoding the string `a b&c=1`:

```
'a' → safe, unchanged → a
' ' (space, 0x20) → %20
'b' → safe, unchanged → b
'&' (0x26) → %26
'c' → safe, unchanged → c
'=' (0x3D) → %3D
'1' → safe, unchanged → 1

Result: "a b&c=1" → "a%20b%26c%3D1"
```

This is precisely why a query-string parameter value containing `&` or `=` must be percent-encoded before being appended to a URL — otherwise the raw `&`/`=` characters would be parsed as delimiters between separate parameters rather than as literal data, which is the root mechanism behind several classes of parameter-injection and request-smuggling bugs.

### UTF-8 (RFC 3629)

Text is not inherently limited to the 128 characters of ASCII. **Unicode**, maintained by the Unicode Consortium, assigns every character in every supported writing system a unique **code point** (e.g., the Euro sign `€` is code point U+20AC). **UTF-8**, standardised in RFC 3629, is the dominant encoding used to represent those code points as a sequence of 1 to 4 bytes, and it is backward-compatible with ASCII: any byte in the ASCII range (0x00–0x7F) is encoded as itself, in a single byte.

UTF-8 uses the leading bits of the first byte to signal how many bytes the character occupies:

| Code point range | Byte pattern | Bytes used |
|---|---|---|
| U+0000 – U+007F | `0xxxxxxx` | 1 |
| U+0080 – U+07FF | `110xxxxx 10xxxxxx` | 2 |
| U+0800 – U+FFFF | `1110xxxx 10xxxxxx 10xxxxxx` | 3 |
| U+10000 – U+10FFFF | `11110xxx 10xxxxxx 10xxxxxx 10xxxxxx` | 4 |

Worked example — encoding the Euro sign, `€` (U+20AC), which falls in the 3-byte range:

```
U+20AC in binary (16 bits): 0010 0000 1010 1100

Split those 16 bits into the 4-bit / 6-bit / 6-bit groups the 3-byte
pattern requires:
  xxxx   = 0010
  xxxxxx = 000010
  xxxxxx = 101100

Fill the template  1110xxxx 10xxxxxx 10xxxxxx :
  Byte 1: 1110 0010 = 0xE2
  Byte 2: 10 000010 = 0x82
  Byte 3: 10 101100 = 0xAC

Result: '€' → bytes E2 82 AC
```

Every continuation byte starts with the fixed bit pattern `10`, which lets a decoder resynchronize to a character boundary even if it starts reading in the middle of a multi-byte sequence — a deliberate self-synchronizing design choice documented in RFC 3629.

### Why encoding is not encryption

This is the single most important conceptual point in this room, and one of the most common misconceptions in security: **encoding is fully reversible by anyone, with no secret required.** Base64, URL-encoding, and UTF-8 all use publicly documented, fixed algorithms — decoding them requires no key, no password, and no computation beyond running the same transformation in reverse. **Encryption**, by contrast, is specifically designed so that recovering the original data (without the correct key) is computationally infeasible. Encoding provides zero confidentiality; its only purposes are compatibility (fitting binary data into a text-safe channel) and correctness (making sure special characters aren't misinterpreted by a parser). Any credential, token, or secret that is "protected" only by Base64 or URL-encoding is, from a security standpoint, protected by nothing at all.

## Why It Matters for Security

- **Malware and exploit obfuscation** routinely Base64-encode payloads (shellcode, PowerShell commands, C2 configuration) not for confidentiality but to survive transport through channels that mangle binary data or certain special characters — analysts decode these trivially, which is exactly why more advanced payloads layer actual encryption or compression on top of encoding.
- **Injection payload construction** (SQL injection, XSS, command injection) frequently relies on URL-encoding or double-encoding to smuggle characters like `'`, `<`, or `;` past naive input filters that only check for the literal, un-decoded character.
- **Data exfiltration detection** — security tooling that inspects network traffic or logs for suspicious patterns must account for the fact that Base64-encoded data looks like random-ish alphanumeric text, and analysts and detection rules alike need to recognize Base64 "shape" (character set, padding, length divisible by 4) to flag it for decoding and inspection.
- **Character encoding mismatches** between what a web application expects (UTF-8) and what it actually receives can enable encoding-based filter bypasses — historically, inconsistent handling of multi-byte and overlong UTF-8 sequences has been used to bypass input validation that assumed single-byte characters.

## Common Pitfalls / Misconfigurations

- **Storing or transmitting secrets Base64-encoded and calling it "encrypted"** — a startlingly common real-world mistake (API keys, credentials, or session data placed in a cookie or config file as Base64 alone), since anyone who intercepts the encoded value can decode it instantly with a one-line command.
- **Failing to URL-decode input consistently before validation**, allowing an attacker to bypass a filter by percent-encoding the disallowed characters, then relying on a downstream component decoding them after the security check has already passed.
- **Double-encoding confusion** — encoding a value twice (or decoding it twice) produces a different, often unexpected result, which both attackers and defenders exploit or trip over respectively.
- **Assuming Base64 output is "safe" for direct inclusion in a URL** without accounting for the fact that standard Base64's `+` and `/` characters are not URL-safe and must be swapped for `-` and `_` (the "Base64URL" variant defined in RFC 4648 §5) before use in a URL or filename.
- **Mixing up character encoding and text encoding entirely** — assuming a byte stream is ASCII when it is actually UTF-8 (or another encoding) can corrupt multi-byte characters and, in security contexts, can also produce inconsistent validation results between components that disagree about the encoding in use.

## Related TryHackMe Rooms in This Series

- [Data Representation](../data-representation/README.md) — the binary/hex/decimal foundation that Base64, URL-encoding, and UTF-8 are all built on top of.
- [Client-Server Basics](../../fundamentals/client-server-basics/README.md) — encodings like URL-encoding exist specifically because of constraints in the HTTP request/URL format used between clients and servers.
- [Inside a Computer System](../../fundamentals/inside-a-computer-system/README.md) — the byte-level storage these encodings ultimately manipulate.

## References

- IETF, RFC 4648, "The Base16, Base32, and Base64 Data Encodings" — https://www.rfc-editor.org/rfc/rfc4648.html
- IETF, RFC 3986, "Uniform Resource Identifier (URI): Generic Syntax" — https://www.rfc-editor.org/rfc/rfc3986.html
- IETF, RFC 3629, "UTF-8, a transformation format of ISO 10646" — https://www.rfc-editor.org/rfc/rfc3629.html
- The Unicode Consortium, "The Unicode Standard" — https://www.unicode.org/standard/standard.html
- Mozilla Developer Network, "Percent-encoding" — https://developer.mozilla.org/en-US/docs/Glossary/Percent-encoding
- Mozilla Developer Network, "Base64" — https://developer.mozilla.org/en-US/docs/Glossary/Base64
- OWASP, "Testing for Stored Encoded Injection" / OWASP Web Security Testing Guide — https://owasp.org/www-project-web-security-testing-guide/
- MITRE, CWE-838: Inappropriate Encoding for Output Context — https://cwe.mitre.org/data/definitions/838.html
