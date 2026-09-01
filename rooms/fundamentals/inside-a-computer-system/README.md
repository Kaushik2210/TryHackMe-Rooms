# Inside a Computer System

**Platform:** TryHackMe · **Type:** Concept Guide (no personal run captured — see note below)
**Primary domain:** Computing Fundamentals

> **Note:** This is a concept guide covering the material this TryHackMe room teaches, written from
> public documentation and reference material. It is not a personal walkthrough — no session evidence
> (screenshots, command output, room-specific answers) is included.

## Overview

Every exploit, every malware sample, and every defensive control ultimately runs on physical hardware built around a small set of components: a processor that executes instructions, memory that holds data temporarily, storage that holds data permanently, and a motherboard that wires them together. This room-level topic covers those building blocks and the **von Neumann architecture** that describes how they cooperate. A working mental model of "what actually happens inside the box" underpins topics as varied as buffer overflows, cold-boot attacks, and firmware-level persistence, all of which exploit the gap between how a system is supposed to behave and what its hardware actually does.

## Core Concepts

### The core components

- **CPU (Central Processing Unit)** — the processor that fetches, decodes, and executes instructions. It contains an **Arithmetic Logic Unit (ALU)** for calculations and logic operations, a **Control Unit** that directs the fetch-decode-execute cycle, and small, extremely fast storage locations called **registers** that hold the data currently being operated on. CPU speed is commonly quoted in clock frequency (GHz), but modern performance also depends heavily on the number of cores, cache size, and instruction-set efficiency.
- **RAM (Random Access Memory)** — volatile, high-speed memory that holds the operating system, running programs, and their working data. "Volatile" means its contents are lost when power is removed, which is precisely why forensic techniques like RAM acquisition must capture memory *before* a machine is powered off, and why cold-boot attacks exploit the fact that DRAM contents fade gradually rather than vanishing instantly.
- **Storage** — non-volatile memory that persists data without power: traditional spinning **Hard Disk Drives (HDDs)**, or modern **Solid-State Drives (SSDs)** built on NAND flash memory, which have no moving parts and are substantially faster for random access. Storage is orders of magnitude slower than RAM, which is why operating systems load active data into RAM and only write back to storage when necessary.
- **Motherboard** — the main printed circuit board that physically connects the CPU, RAM, storage, and peripherals via buses, and hosts the **chipset** that manages communication between components. It also holds the firmware.
- **Firmware / BIOS / UEFI** — low-level software stored in non-volatile memory on the motherboard that initialises hardware and loads the operating system at boot time. Modern systems predominantly use **UEFI (Unified Extensible Firmware Interface)**, which replaced the legacy BIOS standard and adds features such as Secure Boot, larger disk support, and a richer pre-boot environment — but is itself a meaningful attack surface, since firmware-level implants can persist across OS reinstalls.

### The von Neumann architecture

Most general-purpose computers, from smartphones to servers, are built on the **von Neumann architecture**, first described by John von Neumann in his 1945 "First Draft of a Report on the EDVAC." Its defining characteristic is that program instructions and the data they operate on are stored in the *same* memory space and are indistinguishable to the hardware except by context. The CPU repeatedly performs a **fetch-decode-execute cycle**: it fetches the next instruction from memory (using a program counter to track position), decodes what operation it specifies, executes it (potentially reading or writing data, also in memory), and then advances to the next instruction.

This shared-memory design is efficient and flexible, but it is also the structural reason **memory corruption vulnerabilities are possible at all**: because code and data live in the same addressable memory, a bug that lets an attacker write attacker-controlled data into a region that is later interpreted as executable instructions (or that overwrites a return address, redirecting the instruction pointer) can result in arbitrary code execution. Defences such as **Data Execution Prevention (DEP/NX)**, which marks memory pages as either writable or executable but not both, and **Address Space Layout Randomization (ASLR)**, which randomises where code and data are placed in memory, exist specifically to make this class of attack harder — but they are mitigations layered on top of an architecture that fundamentally does not separate code from data.

### Buses and the fetch-decode-execute cycle in practice

Components communicate over **buses** — sets of electrical pathways for address, data, and control signals. The **address bus** carries the memory location being accessed, the **data bus** carries the actual value being read or written, and the **control bus** carries signals coordinating the operation (read vs. write, timing, interrupts). During each fetch-decode-execute cycle, the CPU places an address on the address bus, asserts a read signal on the control bus, and receives the instruction or data over the data bus. Widening these buses (e.g., 32-bit vs. 64-bit) increases how much data or how large an address space can be handled per cycle, which is why the shift from 32-bit to 64-bit computing dramatically raised the maximum addressable memory a system could use.

## Why It Matters for Security

- **Memory forensics** relies directly on understanding volatility: RAM must be imaged live, and tools that dump memory (e.g., for later analysis with the Volatility Framework) exploit the fact that running processes, decrypted keys, and malware artefacts often exist only in RAM, never touching disk.
- **Firmware and boot-chain attacks** target the motherboard/UEFI layer specifically because it executes before the operating system and its own security controls (antivirus, EDR) are active, making firmware implants extremely persistent and hard to detect from within the OS.
- **Memory corruption exploitation** (stack/heap buffer overflows, use-after-free) is only possible because of the shared code/data memory model inherent to von Neumann-style architectures; understanding the fetch-decode-execute cycle and register usage is a prerequisite for understanding how a return address gets overwritten or how shellcode gets executed.
- **Side-channel and physical attacks** — such as cold-boot attacks against RAM remanence, or DMA (Direct Memory Access) attacks over Thunderbolt/FireWire ports that bypass the CPU to read memory directly — depend on the physical properties of these exact components.

## Common Pitfalls / Misconfigurations

- **Assuming data is gone once a machine is powered off** — DRAM does not lose its contents instantaneously, and residual data can sometimes be recovered for a short window after power loss, which is why full-disk encryption keys held only in RAM are a genuine forensic concern.
- **Leaving DEP/NX or ASLR disabled** (or compiling software without stack canaries or position-independent code) removes cheap, well-understood mitigations against the very memory-corruption classes the von Neumann model makes possible.
- **Neglecting firmware updates** — treating UEFI/BIOS as "set and forget" leaves known firmware vulnerabilities unpatched, even when the operating system itself is fully up to date.
- **Underestimating physical access risk** — because RAM, storage, and firmware are physical components, an attacker with brief physical access can potentially extract disk contents, dump RAM, or flash malicious firmware in ways no purely network-based control can prevent.
- **Conflating storage and memory performance characteristics** — assuming an SSD behaves like RAM (or vice versa) leads to incorrect assumptions about data persistence, wear characteristics, and forensic recoverability.

## Related TryHackMe Rooms in This Series

- [Computer Types](../computer-types/README.md) — how these same components are arranged differently across desktops, servers, mobile devices, and embedded systems.
- [Client-Server Basics](../client-server-basics/README.md) — the software-level model that runs on top of this hardware.
- [Data Representation](../../easy/data-representation/README.md) — how the bits held in these components actually represent numbers, text, and instructions.
- See also [Virtualisation Basics](../virtualisation-basics/README.md) for how a hypervisor virtualises the exact CPU/RAM/storage resources described here.

## References

- John von Neumann, "First Draft of a Report on the EDVAC," 1945 (reprinted, IEEE Annals of the History of Computing, 1993) — https://ieeexplore.ieee.org/document/238389
- Intel, "What Does a CPU Do?" — https://www.intel.com/content/www/us/en/gaming/resources/what-does-a-cpu-do.html
- Microsoft, "UEFI firmware overview" — https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/uefi-firmware-overview
- UEFI Forum, "UEFI Specifications" — https://uefi.org/specifications
- Microsoft, "Data Execution Prevention" — https://learn.microsoft.com/en-us/windows/win32/memory/data-execution-prevention
- Microsoft, "Address space layout randomization (ASLR) exploit protection" — https://learn.microsoft.com/en-us/windows/win32/secbp/address-space-layout-randomization
- Volatility Foundation, "The Volatility Framework" — https://www.volatilityfoundation.org/
- Halderman et al., "Lest We Remember: Cold-Boot Attacks on Encryption Keys," USENIX Security 2008 — https://www.usenix.org/legacy/event/sec08/tech/full_papers/halderman/halderman.pdf
