# Computer Types

**Platform:** TryHackMe · **Type:** Concept Guide (no personal run captured — see note below)
**Primary domain:** Computing Fundamentals

> **Note:** This is a concept guide covering the material this TryHackMe room teaches, written from
> public documentation and reference material. It is not a personal walkthrough — no session evidence
> (screenshots, command output, room-specific answers) is included.

## Overview

Not every computer is a laptop or a server. Desktops, laptops, servers, embedded systems, and mobile devices are all built on the same fundamental von Neumann principles (a CPU, memory, and storage cooperating via buses), but they differ enormously in form factor, resource constraints, intended usage pattern, and — critically for security work — attack surface and typical exposure to physical or network access. Recognising which category a target device falls into is often the first step in threat modelling it, because the realistic attack paths against an internet-facing server, an air-gapped industrial embedded controller, and a personal smartphone are very different.

## Core Concepts

### Desktop computers

Desktops are stationary, general-purpose machines with separable components (case, monitor, keyboard, mouse) and typically the most generous power, cooling, and expansion budget of any consumer category. This means desktops usually have the most headroom for upgrades and the most physical ports (multiple USB, PCIe expansion slots), which is convenient but also expands the physical attack surface — more exposed ports mean more avenues for USB-based attacks (malicious HID devices, "rubber duckies," or DMA-capable expansion cards).

### Laptops

Laptops integrate the same core components into a portable, battery-powered chassis with an integrated display and input devices. Portability is the defining trade-off: a laptop is far more likely to be lost, stolen, or used outside a controlled network perimeter than a desktop, which is precisely why full-disk encryption (BitLocker, FileVault, LUKS), device-tracking, and remote-wipe capability are considered baseline controls for laptop fleets in a way they typically are not for stationary desktops.

### Servers

Servers are computers designed to provide services to other computers (clients) over a network, typically optimised for reliability, uptime, and throughput rather than for interactive, single-user use. Server hardware commonly features redundant power supplies, Error-Correcting Code (ECC) memory (which detects and corrects single-bit memory errors to reduce silent data corruption), hot-swappable drives, and support for running many concurrent processes or virtual machines. Because servers are frequently exposed to a network — sometimes directly to the internet — and hold data or run services many clients depend on, they are disproportionately high-value targets: a single compromised server can expose every client that depends on it, unlike compromising one individual desktop or laptop.

### Embedded systems

An embedded system is a computer built into a larger device to perform a specific, often narrowly-scoped function, rather than to run arbitrary general-purpose software — examples include the controller in a smart thermostat, a router's firmware, an ATM, a medical device, or an industrial Programmable Logic Controller (PLC). Embedded systems are frequently resource-constrained (limited CPU, memory, and storage) and are often deployed for very long service lifetimes with infrequent or no patching, which makes them a persistently attractive target: legacy, unpatched embedded devices with known vulnerabilities can remain in production for years after a fix is published, precisely because updating them is operationally difficult or the vendor has stopped supporting them. Embedded systems in industrial contexts (Industrial Control Systems / SCADA) carry particularly severe consequences when compromised, since they can control physical processes.

### Mobile devices

Smartphones and tablets are highly integrated, battery-powered, general-purpose computers built around a System-on-Chip (SoC) that combines the CPU, GPU, memory controller, and often cellular radio onto a single chip. Mobile operating systems (iOS, Android) enforce stronger application sandboxing by default than most desktop operating systems, restricting what any single app can access without explicit permission grants. Mobile devices also introduce attack surfaces unique to their form factor: cellular baseband processors, Bluetooth and NFC radios, app-store supply-chain risk, and a strong tendency for users to install third-party applications from a much wider range of sources and permission levels than on managed corporate desktops.

### A comparative view

| Device type | Typical exposure | Physical access risk | Patch cadence | Notable attack surface |
|---|---|---|---|---|
| Desktop | Internal network, sometimes internet | Low (usually in a fixed, sometimes secured location) | Regular | USB ports, expansion cards |
| Laptop | Internal network, public/untrusted networks | High (portable, frequently lost/stolen) | Regular | Physical theft, Wi-Fi/Bluetooth, boot-level attacks |
| Server | Internal network, often internet-facing | Low-to-moderate (data centre or server room) | Regular, but disruption-averse | Network services, remote administration interfaces |
| Embedded system | Isolated, internal, or internet-facing (IoT) | Variable, sometimes unsupervised public locations | Rare or none | Firmware, default credentials, unpatched known CVEs |
| Mobile device | Cellular, Wi-Fi, Bluetooth, app stores | High (small, portable, frequently used off-network) | Frequent (OS), variable (apps) | App sandbox escapes, baseband, third-party app risk |

## Why It Matters for Security

- **Threat modelling starts with device classification.** A penetration test scope that includes servers, laptops, and embedded IoT devices requires different tooling and different assumptions for each: server assessments focus on exposed network services, laptop assessments consider physical loss and endpoint protection bypass, and embedded/IoT assessments often focus on firmware extraction, default credentials, and hardware interfaces like UART or JTAG.
- **Patch management strategy differs by category.** Desktops and laptops in a managed fleet can generally be patched on a predictable cadence; embedded and IoT devices frequently cannot, which is why network segmentation (isolating embedded/OT devices from the general network) is a standard compensating control rather than an optional extra.
- **Physical security requirements scale with portability.** The likelihood of a device being physically lost, stolen, or tampered with directly informs whether full-disk encryption, secure boot, and remote-wipe are mandatory controls.
- **The "attack surface" of a device type is a security-relevant property in its own right** — more exposed interfaces (network services, radios, physical ports) generally correlate with more potential entry points, independent of any specific vulnerability.

## Common Pitfalls / Misconfigurations

- **Treating all endpoints as equivalent for patching and monitoring purposes** — applying a single "patch every 30 days" policy without accounting for embedded/OT devices that cannot be patched without a planned outage (or at all) leads to those devices silently accumulating known vulnerabilities.
- **Leaving default credentials on embedded/IoT devices** — one of the most consistently exploited weaknesses across router, camera, and industrial-controller compromises is simply that default administrative credentials were never changed.
- **Failing to encrypt portable devices** — laptops and mobile devices without full-disk encryption turn a simple physical theft into a full data breach.
- **Assuming servers are inherently more secure because they are "professional" hardware** — server-grade hardware improves reliability, not security; a misconfigured, internet-facing server is still exactly as exploitable as a misconfigured desktop.
- **Ignoring network segmentation for embedded systems** — placing IoT or OT devices on the same flat network as general-purpose corporate endpoints means a compromised smart device can become a pivot point into far more sensitive systems.

## Related TryHackMe Rooms in This Series

- [Inside a Computer System](../inside-a-computer-system/README.md) — the shared hardware building blocks (CPU, RAM, storage, motherboard) referenced throughout this comparison.
- [Client-Server Basics](../client-server-basics/README.md) — servers as a device category are also a logical role in the client-server model.
- See also [Virtualisation Basics](../virtualisation-basics/README.md) for how server hardware is commonly subdivided into virtual machines rather than dedicated per-service.

## References

- NIST SP 800-124 Rev. 2, "Guidelines for Managing the Security of Mobile Devices in the Enterprise" — https://csrc.nist.gov/pubs/sp/800/124/r2/final
- NIST SP 800-82 Rev. 3, "Guide to Operational Technology (OT) Security" — https://csrc.nist.gov/pubs/sp/800/82/r3/final
- CISA, "Industrial Control Systems" — https://www.cisa.gov/topics/industrial-control-systems
- Android Open Source Project, "Application Sandbox" — https://source.android.com/docs/security/app-sandbox
- Apple Platform Security Guide — https://support.apple.com/guide/security/welcome/web
- Microsoft, "BitLocker overview" — https://learn.microsoft.com/en-us/windows/security/operating-system-security/data-protection/bitlocker/
- CISA, "Internet of Things (IoT)" — https://www.cisa.gov/topics/cyber-threats-and-advisories/internet-things
