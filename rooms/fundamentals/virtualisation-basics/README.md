# Virtualisation Basics

**Platform:** TryHackMe · **Type:** Concept Guide (no personal run captured — see note below)
**Primary domain:** Computing Fundamentals

> **Note:** This is a concept guide covering the material this TryHackMe room teaches, written from
> public documentation and reference material. It is not a personal walkthrough — no session evidence
> (screenshots, command output, room-specific answers) is included.

## Overview

Virtualisation is the technique of running one or more simulated ("virtual") computers on top of a single piece of physical hardware. It is the foundation almost every cybersecurity lab, TryHackMe machine, and enterprise data centre relies on: instead of provisioning a dedicated physical machine for every task, a **hypervisor** carves up one host's CPU, memory, storage, and network resources into isolated virtual machines (VMs). TryHackMe's "Virtualisation Basics" room introduces this concept because nearly every subsequent room — and most real-world offensive security work — happens inside a VM. Understanding what a hypervisor actually does, and where its isolation boundaries are, is a prerequisite for reasoning about lab safety, snapshotting, and VM-escape risk.

## Core Concepts

### What a hypervisor does

A hypervisor (also called a Virtual Machine Monitor, or VMM) is the software layer that creates and manages virtual machines. It presents each VM with virtualised CPU, RAM, disk, and network interfaces, and schedules access to the real underlying hardware so that multiple guest operating systems can run concurrently without being aware of one another. The concept was formalised academically by Gerald Popek and Robert Goldberg in their 1974 paper "Formal Requirements for Virtualizable Third Generation Architectures," which defined the properties (equivalence, resource control, efficiency) a system must have to support virtualisation cleanly.

### Type-1 vs. Type-2 hypervisors

Hypervisors are conventionally split into two categories:

| Type | Also called | Runs on | Examples | Typical use |
|---|---|---|---|---|
| Type-1 | "Bare-metal" | Directly on physical hardware, with no host OS underneath | VMware ESXi, Microsoft Hyper-V (server role), Citrix Hypervisor, KVM (as part of the Linux kernel) | Data centres, cloud providers, production server virtualisation |
| Type-2 | "Hosted" | On top of a conventional host operating system | Oracle VirtualBox, VMware Workstation/Fusion, Parallels Desktop | Developer workstations, security labs, personal test environments |

A Type-1 hypervisor has direct access to hardware, so it typically achieves lower overhead and stronger isolation — this is why cloud platforms (AWS, Azure, GCP) and most enterprise virtualisation run Type-1 hypervisors such as VMware ESXi or KVM. A Type-2 hypervisor is itself an application running under a general-purpose OS (Windows, macOS, Linux), which makes it far easier to install and use for a home lab, but adds an extra software layer between the VM and the hardware, and inherits any weaknesses of the host OS.

### Hardware-assisted virtualisation

Modern CPUs include dedicated instruction set extensions — Intel VT-x and AMD-V — that let a hypervisor trap and handle privileged instructions from a guest OS efficiently, instead of relying purely on slower software techniques like binary translation. Without these extensions (or with them disabled in firmware/BIOS), many hypervisors either fail to start 64-bit guests or fall back to much slower emulation. TryHackMe and most cloud-lab providers require nested virtualisation support (running a hypervisor inside an already-virtualised cloud instance) for exactly this reason.

### Full virtualisation, paravirtualisation, and containers

- **Full virtualisation** presents a complete virtual hardware platform to the guest; the guest OS is unmodified and unaware it is virtualised (VMware, VirtualBox, Hyper-V, KVM).
- **Paravirtualisation** requires the guest OS to be modified to call hypervisor-aware APIs directly (historically used by early Xen), trading compatibility for performance.
- **OS-level virtualisation / containers** (Docker, LXC) do not virtualise hardware at all — they isolate processes sharing a single kernel using Linux namespaces and cgroups. Containers are lighter weight than VMs but offer weaker isolation because a kernel vulnerability can potentially be exploited to escape a container, whereas escaping a VM typically requires a hypervisor-level vulnerability.

### Key VM building blocks

- **Virtual disk** — a file (`.vmdk`, `.vdi`, `.vhdx`, `.qcow2`) on the host filesystem that the hypervisor presents to the guest as a physical disk.
- **Virtual NIC** — a software network interface, which the hypervisor can connect in different modes: **NAT** (the VM shares the host's IP and appears as one device to the outside network), **Bridged** (the VM gets its own address on the physical LAN, appearing as a separate device), **Host-only** (an isolated virtual network reachable only from the host), or **Internal Network** (isolated even from the host, used for offensive labs where VM-to-VM traffic must never reach the real network).
- **Snapshots** — a saved point-in-time state of a VM's disk (and optionally memory) that can be reverted to instantly. This is what makes VMs ideal for security testing: a lab machine can be deliberately compromised, examined, and then reverted to a clean baseline in seconds, at essentially zero cost compared to reimaging physical hardware.

## Why It Matters for Security

Virtualisation underpins nearly all practical offensive-security work:

- **Isolation for malware analysis and exploitation** — running untrusted binaries or exploit payloads inside a VM (ideally on an isolated internal or host-only network) keeps the attack contained if something goes wrong, and TryHackMe's own attack boxes are themselves VMs spun up per user.
- **Disposable, repeatable environments** — a snapshot lets an analyst return a compromised VM to a known-good state instantly, which is essential for both malware analysis and CTF-style rooms where the same target needs to be attacked repeatedly from scratch.
- **VM escape** is itself a serious vulnerability class: a flaw in the hypervisor that lets code inside a guest break out and execute on the host or attack sibling VMs. Historical examples include VMware's CVE-2017-4901 and various Xen/QEMU escape bugs; cloud providers and hypervisor vendors treat these as critical because a successful escape can compromise every tenant on a shared physical host.
- **Nested virtualisation and lab safety** — understanding hypervisor networking modes (NAT vs. bridged vs. host-only) directly determines whether an attack lab is actually isolated from the analyst's real network, or accidentally exposed to it.

## Common Pitfalls / Misconfigurations

- **Bridged networking on an offensive lab VM** — accidentally putting an "attack box" on bridged mode exposes it directly to the local network, which can leak scanning traffic or exploit payloads outside the intended lab boundary.
- **Forgetting to revert to a clean snapshot** — reusing a VM that was previously compromised (in a lab exercise) without reverting can leave backdoors or artefacts in place, contaminating later results.
- **Disabling hardware virtualisation extensions** — leaving VT-x/AMD-V disabled in firmware causes confusing failures ("this host supports Intel VT-x, but it is not enabled") that are frequently misdiagnosed as software bugs.
- **Treating containers as equivalent to VMs for isolation** — assuming a Docker container provides the same security boundary as a full VM is a common and dangerous misconception, since containers share the host kernel.
- **Under-resourcing nested labs** — running multiple VMs (attacker + multiple targets) on a host with insufficient RAM/CPU leads to instability that is easy to mistake for a networking or exploitation problem.

## Related TryHackMe Rooms in This Series

- [Inside a Computer System](../inside-a-computer-system/README.md) — covers the physical CPU/RAM/storage resources a hypervisor virtualises.
- [Computer Types](../computer-types/README.md) — contrasts the physical device categories that host or run as VMs.
- [Client-Server Basics](../client-server-basics/README.md) — many virtualised labs are themselves client-server setups (attack box vs. target).

## References

- VMware, "What is a Hypervisor?" — https://www.vmware.com/topics/hypervisor
- VMware ESXi documentation — https://docs.vmware.com/en/VMware-vSphere/index.html
- Oracle, VirtualBox User Manual — https://www.virtualbox.org/manual/
- Microsoft, "Introduction to Hyper-V on Windows" — https://learn.microsoft.com/en-us/virtualization/hyper-v-on-windows/about/
- G. Popek and R. Goldberg, "Formal Requirements for Virtualizable Third Generation Architectures," Communications of the ACM, 1974 — https://dl.acm.org/doi/10.1145/361011.361073
- Intel, "Intel Virtualization Technology (VT-x)" — https://www.intel.com/content/www/us/en/virtualization/virtualization-technology/intel-virtualization-technology.html
- Red Hat, "What is a hypervisor?" — https://www.redhat.com/en/topics/virtualization/what-is-a-hypervisor
- Red Hat, "Containers vs. virtual machines" — https://www.redhat.com/en/topics/containers/containers-vs-vms
- NIST SP 800-125, "Guide to Security for Full Virtualization Technologies" — https://csrc.nist.gov/pubs/sp/800/125/final
