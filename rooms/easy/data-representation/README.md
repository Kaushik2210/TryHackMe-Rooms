# Data Representation

**Platform:** TryHackMe · **Type:** Concept Guide (no personal run captured — see note below)
**Primary domain:** Computing Fundamentals

> **Note:** This is a concept guide covering the material this TryHackMe room teaches, written from
> public documentation and reference material. It is not a personal walkthrough — no session evidence
> (screenshots, command output, room-specific answers) is included.

## Overview

Computers only ever store and manipulate **bits** — binary digits that are either 0 or 1. Every number, character, image, and instruction a computer processes is ultimately some arrangement of bits, interpreted according to an agreed-upon convention. This room-level topic covers how those bits are grouped and interpreted as numbers (binary, hexadecimal, decimal), how negative numbers are represented (two's complement), and how fractional numbers are represented (IEEE 754 floating point). These conventions are not abstract trivia: they are the literal substrate that vulnerability classes like integer overflow, sign-extension bugs, and floating-point precision errors exploit.

## Core Concepts

### Binary, decimal, and hexadecimal

A **bit** is a single binary digit (0 or 1). A group of 8 bits is a **byte**, which can represent 2⁸ = 256 distinct values (0–255 unsigned). Because raw binary is long and error-prone for humans to read, **hexadecimal (base 16)** is used as a compact, exact shorthand: each hex digit (0–9, then A–F for 10–15) represents exactly 4 bits (one "nibble"), so a byte is always exactly two hex digits.

Worked example — converting the decimal number **202** to binary and hex:

```
202 in binary (powers of 2: 128 64 32 16 8 4 2 1):
  128 + 64 + 8 + 2 = 202
  bit pattern:        1  1  0  0  1  0  1  0
  → 202 (decimal) = 11001010 (binary)

Grouping into nibbles for hex:
  1100 = C     1010 = A
  → 11001010 (binary) = CA (hexadecimal)
```

So `202` (decimal) = `11001010` (binary) = `0xCA` (hex) — three notations for the exact same bit pattern. This is why hex is used pervasively in security tooling (memory dumps, disassembly, IP addressing in IPv6, colour codes, hash digests): it is a lossless, compact way to write out raw bytes.

### Unsigned vs. signed integers, and two's complement

An unsigned N-bit value can represent 0 to 2^N − 1. To represent negative numbers, essentially all modern systems use **two's complement** representation. To find the two's complement (negative) representation of a positive number: invert every bit, then add 1.

Worked example — representing **−45** as an 8-bit two's complement value:

```
45 in binary:            00101101
Invert every bit:        11010010
Add 1:                   11010011   ← this is −45 in 8-bit two's complement

Check: 11010011 as an unsigned 8-bit value = 128+64+16+2+1 = 211
       211 − 256 (2^8) = −45   ✓
```

Two's complement is used because it lets a CPU perform subtraction using the same binary adder circuitry as addition (subtracting a number is the same as adding its two's complement), and because it has a single representation of zero, unlike sign-magnitude or one's-complement schemes. The **most significant bit (MSB)** acts as the sign bit: 0 for non-negative, 1 for negative — but it is not simply a sign flag layered on top of the magnitude, which is a common source of confusion and of real bugs (an unsigned integer that overflows past its maximum value wraps around to 0, while a signed integer that overflows its maximum positive value wraps around to the most negative representable value).

### IEEE 754 floating point

Fractional numbers are represented using the **IEEE 754** standard for floating-point arithmetic (current edition: IEEE 754-2019), which defines formats including 32-bit **single precision** and 64-bit **double precision**. A single-precision float is laid out as:

| Field | Bits | Purpose |
|---|---|---|
| Sign | 1 bit | 0 = positive, 1 = negative |
| Exponent | 8 bits | Stored as the true exponent + a bias of 127 |
| Mantissa (fraction) | 23 bits | The fractional part of a normalized `1.xxxxx × 2^exponent` value |

Worked example — encoding **−13.25** as a 32-bit IEEE 754 float:

```
1. Convert 13.25 to binary:
   13 = 1101, 0.25 = 0.01  →  13.25 = 1101.01

2. Normalize to 1.mantissa × 2^exponent form:
   1101.01 = 1.10101 × 2^3

3. Sign bit: negative → 1

4. Exponent: true exponent 3, biased by 127 → 3 + 127 = 130
   130 in 8-bit binary = 10000010

5. Mantissa: take the bits after the leading "1." and pad to 23 bits:
   10101 → 10101000000000000000000

Full 32-bit layout:
   sign     exponent    mantissa
   1        10000010    10101000000000000000000

As hex (grouped into nibbles): 0xC1540000
```

This same layout is why floating-point numbers cannot represent every decimal fraction exactly — 0.1, for instance, has no exact finite binary fraction representation, which is why floating-point comparisons for equality (`0.1 + 0.2 == 0.3`) are notoriously unreliable across virtually every programming language, and why financial or security-critical calculations generally use fixed-point or arbitrary-precision decimal types instead of `float`/`double`.

### Character encoding as a form of data representation

Text is also, at the lowest level, just numbers interpreted as characters via a mapping such as ASCII (7-bit, values 0–127) or the far larger Unicode standard (which the Data Encoding room and UTF-8 build on). The letter `A`, for example, is simply the byte value 65 (`0x41`, `01000001`), interpreted as a character only because the display/application layer agrees to interpret it that way — the underlying bits are identical to however else that same byte pattern might be interpreted (as a small integer, as part of an instruction, and so on).

## Why It Matters for Security

- **Integer overflow/underflow vulnerabilities** occur exactly at the representational boundaries described above: an unsigned 8-bit counter incremented past 255 wraps to 0, and a signed 32-bit value incremented past its maximum wraps to a large negative number — both of which have caused real-world exploitable bugs (buffer size miscalculations, bypassed bounds checks) when developers assumed arithmetic would never overflow.
- **Type confusion and sign-extension bugs** arise when a value is reinterpreted between signed and unsigned, or between different bit widths, without the developer accounting for how two's complement changes the interpreted value — a classic source of bypassed length checks in memory-unsafe languages like C.
- **Floating-point imprecision** has been exploited or has caused failures in financial systems and safety-critical software; understanding that IEEE 754 floats trade exactness for range is essential when auditing code that performs monetary or safety-relevant calculations.
- **Reading raw memory or disassembly** requires fluency in hex-to-binary-to-decimal conversion, since debuggers, disassemblers, and packet captures overwhelmingly display raw data in hexadecimal.

## Common Pitfalls / Misconfigurations

- **Comparing floats for exact equality** instead of using an epsilon-based tolerance comparison, leading to logic errors that can sometimes be leveraged to bypass checks (e.g., a price or threshold check that never triggers due to floating-point drift).
- **Using signed types for values that should never be negative** (array indices, buffer lengths) without validating the sign, allowing a negative value to be misinterpreted as a very large unsigned length when cast, historically a common root cause of buffer overflow primitives.
- **Assuming a fixed-width integer can never overflow** in code paths that perform arithmetic on user-controlled input, rather than explicitly checking bounds or using checked/saturating arithmetic.
- **Confusing endianness (byte order) with the numeric value itself** — the same bytes read as little-endian vs. big-endian produce different numbers, a frequent source of confusion when reverse-engineering binary formats or network protocols.

## Related TryHackMe Rooms in This Series

- [Data Encoding](../data-encoding/README.md) — builds directly on this room's binary/hex foundation to cover Base64, URL-encoding, and UTF-8.
- [Inside a Computer System](../../fundamentals/inside-a-computer-system/README.md) — the hardware (CPU registers, memory) that stores and operates on these representations.
- [Client-Server Basics](../../fundamentals/client-server-basics/README.md) — data representation underlies how information is serialized and transmitted between client and server.

## References

- IEEE, "754-2019 - IEEE Standard for Floating-Point Arithmetic" — https://ieeexplore.ieee.org/document/8766229
- Wikipedia (community-maintained technical reference), "IEEE 754" — https://en.wikipedia.org/wiki/IEEE_754
- Wikipedia, "Two's complement" — https://en.wikipedia.org/wiki/Two%27s_complement
- Computerphile / University of Nottingham, "Floating Point Numbers" — https://www.youtube.com/watch?v=PZRI1IfStY0
- CMU, "Bits, Bytes and Integers" (15-213 course notes) — https://www.cs.cmu.edu/afs/cs/academic/class/15213-f16/www/lectures/03-bits-ints.pdf
- MITRE, CWE-190: Integer Overflow or Wraparound — https://cwe.mitre.org/data/definitions/190.html
- MITRE, CWE-681: Incorrect Conversion between Numeric Types — https://cwe.mitre.org/data/definitions/681.html
