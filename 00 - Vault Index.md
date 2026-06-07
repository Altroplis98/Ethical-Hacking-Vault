---
tags: [pentest, vault, moc, index]
type: home
aliases: [Home, MOC, Vault Home]
---

# Ethical Hacking Vault

CEH 7-phase methodology + HTB workflow library. Each folder is a phase or reference category; each file inside is a single tool, technique, or concept.

> [!warning] Authorized testing only
> Use against systems you own or have explicit written authorization to test. The "Get-out-of-jail-free" letter rule applies to every command in this vault.

## How this vault is organized

- **Folders 01-08** mirror the ethical hacking lifecycle phases.
- **Folders 09-11** are quick-reference cheatsheets you'll hit constantly.
- **Folder 12** is HTB / lab methodology - walkthroughs and "when you see X, do Y" pattern recognition.

Every folder has a `00 - README.md` that lists the files inside it.

## Lifecycle

1. [[01 - Pre-Engagement/00 - README|Pre-Engagement]] - legal, scope, ROE, infra setup
2. [[02 - Reconnaissance and OSINT/00 - README|Reconnaissance & OSINT]] - passive + active recon
3. [[03 - Scanning/00 - README|Scanning]] - hosts, ports, services
4. [[04 - Enumeration/00 - README|Enumeration]] - protocol interrogation
5. [[05 - Vulnerability Analysis/00 - README|Vulnerability Analysis]] - mapping findings to weaknesses
6. [[06 - Gaining Access/00 - README|Gaining Access]] - exploitation
7. [[07 - Post-Exploitation/Linux/00 - README|Post-Exploitation]] - priv-esc, lateral, persistence
8. [[08 - Tracks and Reporting/00 - README|Tracks & Reporting]] - log-clearing study + report writing

## Quick reference

- [[09 - Service Cheatsheets/00 - README|Service Cheatsheets]] - one file per protocol (FTP/SSH/SMB/MSSQL/etc.)
- [[10 - Wireless/00 - README|Wireless]] - WiFi, BLE, RFID
- [[11 - Shells Transfer Hashes/00 - README|Shells / Transfer / Hashes]] - reverse shells, file transfer, hashcat IDs

## Workflow library

- [[12 - HTB Workflows/00 - README|HTB / Lab Workflows]] - **start here when stuck on a box**

## Common external links

| Resource | Use |
| --- | --- |
| [revshells.com](https://www.revshells.com) | Reverse shell generator |
| [GTFOBins](https://gtfobins.github.io) | Linux priv-esc via SUID/sudo |
| [LOLBAS](https://lolbas-project.github.io) | Windows living-off-the-land binaries |
| [HackTricks](https://book.hacktricks.xyz) | Methodology + technique deep-dives |
| [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings) | Web/OS payload archive |
| [Exploit-DB](https://www.exploit-db.com) | Public exploit search |
| [Nuclei templates](https://github.com/projectdiscovery/nuclei-templates) | Automated CVE/misconfig checks |
| [SecLists](https://github.com/danielmiessler/SecLists) | Wordlists |
| [ired.team](https://www.ired.team) | Red-team technique notes |
| [adsecurity.org](https://adsecurity.org) | AD attacker/defender content |

## CEH phase → MITRE ATT&CK

| Phase | ATT&CK Tactic(s) |
| --- | --- |
| 1. Reconnaissance | TA0043 |
| 2. Scanning | TA0043 (T1595 Active Scanning) |
| 3. Enumeration | TA0007 Discovery |
| 4. Vuln Analysis | TA0042 Resource Development |
| 5. Gaining Access | TA0001 Initial Access, TA0002 Execution, TA0006 Credential Access |
| 6. Post-Exploitation | TA0003 Persistence, TA0004 PrivEsc, TA0008 Lateral, TA0009 Collection, TA0010 Exfil, TA0011 C2 |
| 7. Clearing Tracks | TA0005 Defense Evasion (T1070 Indicator Removal) |

> [!tip] How to use this vault
> - **Ctrl+O** Quick Switcher - jump to any file by name.
> - **Ctrl+G** Graph View - see how everything links together.
> - **Ctrl+Shift+F** Search all - find any command across all notes.
> - When stuck on a box, open [[12 - HTB Workflows/01 - When You See X Do Y|the X→Y tip card]].
