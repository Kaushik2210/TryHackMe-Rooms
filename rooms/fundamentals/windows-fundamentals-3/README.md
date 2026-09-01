# Windows Fundamentals 3

**Platform:** TryHackMe · **Type:** Concept Guide (no personal run captured — see note below)
**Primary domain:** Windows / Operating Systems

> **Note:** This is a concept guide covering the material this TryHackMe room teaches, written from
> public documentation and reference material. It is not a personal walkthrough — no session evidence
> (screenshots, command output from a specific machine, room-specific answers) is included. Command
> examples below are generic reference examples, not captures from a completed session.

## Overview

Windows Fundamentals 3 closes out the series by focusing on the built-in tools Microsoft ships to keep a
Windows device secure: Windows Update, Microsoft Defender Antivirus, Windows Firewall, SmartScreen, the
Trusted Platform Module (TPM), BitLocker drive encryption, and the Volume Shadow Copy Service (VSS). If
Part 1 was "how the system is organized" and Part 2 was "how to configure and inspect it," Part 3 is
"how it defends itself by default" — the baseline every later, more offensively-focused room assumes a
target either has, has partially configured, or has had disabled.

## Core Concepts

### Windows Update

**Windows Update** is the built-in servicing mechanism that delivers security patches, feature updates,
and driver updates. Security-relevant fixes are typically released on **Patch Tuesday** — the second
Tuesday of each month — though Microsoft can and does push out-of-band updates for actively exploited
vulnerabilities on an urgent basis. Update settings expose options for active hours (to avoid
disruptive reboots during work), deferral periods for feature updates, and — on managed/enterprise
deployments — integration with Windows Server Update Services (WSUS) or Windows Update for Business to
centrally control rollout timing.

### Microsoft Defender Antivirus

**Microsoft Defender Antivirus** is the antimalware engine built into Windows since Windows 8, providing:

- **Real-time protection** — continuous scanning of files as they're accessed, created, or modified.
- **Cloud-delivered protection** — queries Microsoft's cloud for near-instant verdicts on suspicious
  files rather than waiting for the next local signature update.
- **Scheduled and on-demand scans** — quick, full, custom, and offline scan types, the last of which
  boots into a minimal pre-OS environment to catch malware that can hide from a running OS.
- **Controlled folder access** — a ransomware-mitigation feature that restricts which applications may
  write to designated protected folders.

Defender's status, along with the rest of the security stack below, is surfaced in the consolidated
**Windows Security** app.

### Windows Firewall

**Windows Defender Firewall** filters inbound and outbound network traffic based on rules, and applies a
different rule set depending on which of three **network profiles** the current connection is
categorized under:

| Profile | Typical use | Default posture |
|---|---|---|
| Domain | Connected to a network with a reachable Active Directory domain controller | Trusts the network more; managed centrally via Group Policy |
| Private | Home or trusted work networks manually marked "private" | Moderate trust — some discovery/sharing enabled |
| Public | Coffee shops, airports, untrusted networks | Most restrictive — discovery and sharing disabled by default |

Rules can be scoped by program, port, protocol, and profile, and both the GUI (`wf.msc`, the "Windows
Defender Firewall with Advanced Security" console) and the `netsh advfirewall` / `New-NetFirewallRule`
command-line interfaces can manage them.

### SmartScreen

**Microsoft Defender SmartScreen** checks downloaded files and visited URLs against a Microsoft-maintained
reputation service, warning or blocking execution/navigation for files and sites with poor or unknown
reputation — a first line of defense against phishing pages and freshly-compiled malware that hasn't
yet been flagged by traditional signatures.

### TPM and BitLocker

The **Trusted Platform Module (TPM)** is a dedicated hardware (or firmware) security chip that generates
and stores cryptographic keys in a way that's isolated from the main OS and resistant to physical
tampering. **BitLocker** is Windows' full-volume encryption feature, and on TPM-equipped hardware it
typically binds the encryption key to the TPM, so the drive unlocks transparently on a trusted,
unmodified boot chain but requires a recovery key if the boot configuration changes unexpectedly (e.g.,
someone tries to boot the drive from different hardware). Without a TPM, BitLocker can still be used
with a USB startup key or a password, at reduced convenience and slightly different threat coverage.

### Windows Security app

The **Windows Security** app is the consolidated dashboard that surfaces the status of every control
covered in this room — Virus & threat protection, Firewall & network protection, App & browser control,
Device security (including TPM status), and Account protection — in one place, rather than requiring an
administrator to open each individual console separately. For a quick health check, this is the single
fastest place to confirm real-time protection is active, the firewall is enabled for the current network
profile, and no pending actions are flagged. On managed enterprise devices, this local view is
frequently overridden or supplemented by centralized policy and telemetry pushed from an endpoint
management platform, which can cause some controls here to appear "managed by your organization" and
non-editable locally.

### Volume Shadow Copy Service (VSS)

**VSS** enables point-in-time snapshots of a volume even while files on it are open and in use, which
underpins Windows' **File History** / "Previous Versions" restore feature and is heavily relied on by
backup software. From a security standpoint, VSS snapshots are frequently both a recovery lifeline
(restoring pre-encryption versions of files after a ransomware attack) and a deliberate target — much
ransomware explicitly deletes shadow copies (commonly via `vssadmin delete shadows` or similar) to
prevent that exact recovery path.

## Why It Matters for Security

- **This room's tools are the actual baseline defenses present on virtually every modern Windows
  endpoint**, so recognizing their expected state (enabled, up to date, correctly configured) is the
  starting point for any assessment of a machine's security posture.
- **Attackers routinely target these controls directly** — disabling Defender via registry or Group
  Policy tampering, deleting shadow copies before deploying ransomware, or abusing SmartScreen bypass
  techniques (e.g., mark-of-the-web stripping) are all well-documented, common techniques.
- **BitLocker and TPM matter enormously for stolen or lost device scenarios** — without full-disk
  encryption, physical access to a powered-off machine is close to game over for data confidentiality,
  regardless of how strong the account password is.

## Common Pitfalls / Misconfigurations

- **Leaving a network incorrectly categorized as Private** when it's actually untrusted, which
  loosens the firewall profile applied and can expose file/printer sharing unnecessarily.
- **Disabling real-time protection "temporarily" and forgetting to re-enable it**, or excluding overly
  broad paths (e.g., an entire user profile) from Defender scanning to work around a false positive.
- **Assuming BitLocker alone protects data if the machine is left unlocked and logged in.** BitLocker
  protects data at rest (drive powered off/locked); it does nothing once Windows has booted and
  decrypted the volume for a logged-in session.
- **Not backing up or escrowing the BitLocker recovery key** (e.g., to Azure AD/Entra ID or Active
  Directory) before enabling encryption, risking permanent data loss if the TPM state changes
  unexpectedly.
- **Ignoring Windows Update deferral settings on critical systems**, leaving known, actively-exploited
  vulnerabilities unpatched far longer than the monthly cadence would suggest is necessary.

## Related TryHackMe Rooms in This Series

1. [Windows Fundamentals 1](../windows-fundamentals-1/README.md)
2. [Windows Fundamentals 2](../windows-fundamentals-2/README.md)
3. Windows Fundamentals 3 *(this room)*
4. [Windows Basics](../../easy/windows-basics/README.md)
5. [Windows CLI Basics](../../easy/windows-cli-basics/README.md)
6. [Windows Command Line](../../easy/windows-command-line/README.md)
7. [Windows PowerShell](../../easy/windows-powershell/README.md)
8. [Active Directory Basics](../../easy/active-directory-basics/README.md)

## References

- [Microsoft Learn: Windows Update overview](https://learn.microsoft.com/en-us/windows/deployment/update/index)
- [Microsoft Learn: Microsoft Defender Antivirus in Windows](https://learn.microsoft.com/en-us/defender-endpoint/microsoft-defender-antivirus-windows)
- [Microsoft Learn: Windows Firewall documentation](https://learn.microsoft.com/en-us/windows/security/operating-system-security/network-security/windows-firewall/)
- [Microsoft Learn: Microsoft Defender SmartScreen overview](https://learn.microsoft.com/en-us/windows/security/operating-system-security/virus-and-threat-protection/microsoft-defender-smartscreen/)
- [Microsoft Learn: Trusted Platform Module (TPM) overview](https://learn.microsoft.com/en-us/windows/security/hardware-security/tpm/trusted-platform-module-overview)
- [Microsoft Learn: BitLocker overview](https://learn.microsoft.com/en-us/windows/security/operating-system-security/data-protection/bitlocker/)
- [Microsoft Learn: Volume Shadow Copy Service](https://learn.microsoft.com/en-us/windows-server/storage/file-server/volume-shadow-copy-service)
