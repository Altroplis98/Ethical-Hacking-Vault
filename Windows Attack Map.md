---
tags: [moc, attack-map, windows, active-directory]
type: map-of-content
---

# Windows Attack Map

Map of Content for **Windows / Active Directory** targets — domain-joined workstations, member servers, and Domain Controllers. Organized by engagement phase. For Linux targets, see [[Linux Attack Map]].

[[00 - Vault Index|Home]] · [[00 - USE ME|How to use this vault]] · [[12 - HTB Workflows/02 - General Methodology|Methodology]]

> [!tip] How to use this MOC
> Click into a phase, scan the one-liners, jump to the note that matches your situation. Don't read top-to-bottom — open this when you need a specific tool/technique and use Ctrl+F.

---

## Phase 1 — Recon (no creds, external-ish)

What kind of Windows machine am I looking at?

- [[12 - HTB Workflows/14 - Machine Fingerprinting by Port Combos]] — port combo → machine role (DC, RODC, workstation, etc.)
- [[12 - HTB Workflows/01 - When You See X Do Y]] — per-service pattern → next move
- [[03 - Scanning/04 - nmap Host Discovery]] — find live Windows hosts (ICMP often filtered)
- [[03 - Scanning/05 - nmap Basics]] — full TCP + scripts
- [[03 - Scanning/07 - nmap Service and Version Detection]] — pin the exact OS/build for CVE work
- [[02 - Reconnaissance and OSINT/15 - Shodan]] — external-facing Windows assets
- [[02 - Reconnaissance and OSINT/17 - CrossLinked]] — LinkedIn → username list for spraying

## Phase 2 — Service Enumeration (no creds yet)

- [[09 - Service Cheatsheets/11 - SMB (445 139)]] — anonymous shares, null sessions, signing check
- [[09 - Service Cheatsheets/10 - LDAP LDAPS (389 636)]] — anonymous bind?
- [[09 - Service Cheatsheets/26 - Kerberos and AD Services (88 464 3268 3269)]] — DC confirmation, GC enum
- [[09 - Service Cheatsheets/17 - RDP (3389)]] — NLA check, BlueKeep candidates
- [[09 - Service Cheatsheets/20 - WinRM (5985 5986)]] — credential-ready remote exec target
- [[09 - Service Cheatsheets/13 - MSSQL (1433)]] — default sa, xp_cmdshell
- [[09 - Service Cheatsheets/25 - IPMI (623)]] — out-of-band server management
- [[04 - Enumeration/01 - enum4linux and enum4linux-ng]] — anonymous SMB/RPC enum
- [[04 - Enumeration/02 - smbclient]] — list shares, anonymous access
- [[04 - Enumeration/03 - smbmap]] — share permission matrix
- [[04 - Enumeration/04 - rpcclient]] — null session → enumdomusers, querydispinfo
- [[04 - Enumeration/05 - NetExec (nxc)]] — multi-protocol enum/spray (replaces CME)
- [[04 - Enumeration/06 - nmap SMB Scripts]] — smb-os-discovery, smb-vuln-*, smb2-security-mode
- [[04 - Enumeration/07 - ldapsearch]] — manual LDAP queries (anon + auth)
- [[04 - Enumeration/08 - windapsearch]] — automated AD enum from one cred
- [[04 - Enumeration/12 - kerbrute]] — pre-auth username validation, password spray

## Phase 3 — Initial Access (foothold)

### AD-specific (no cred needed)

- [[06 - Gaining Access/16 - AS-REP Roasting]] — users without Kerberos pre-auth → crackable hash
- [[06 - Gaining Access/13 - Impacket GetNPUsers (AS-REP)]] — Impacket execution of the above
- [[06 - Gaining Access/11 - Responder LLMNR NBT-NS Poisoning]] — capture NetNTLMv2 from broadcast queries
- [[06 - Gaining Access/22 - Coercion (PetitPotam Coercer)]] — force a target to authenticate to you
- [[06 - Gaining Access/12 - ntlmrelayx]] — relay captured NTLM to SMB/LDAP/HTTP for lateral move

### AD-specific (need 1 valid cred)

- [[06 - Gaining Access/15 - Kerberoasting]] — service account TGS → crack offline
- [[06 - Gaining Access/14 - Impacket GetUserSPNs (Kerberoast)]] — Impacket execution
- [[06 - Gaining Access/04 - NetExec Password Attacks]] — spray a cred set across the network
- [[06 - Gaining Access/05 - Password Spraying Strategy]] — lockout-safe spray plan
- [[04 - Enumeration/09 - BloodHound]] — graph the AD attack path from your foothold cred
- [[04 - Enumeration/10 - SharpHound]] — collector (Windows)
- [[04 - Enumeration/11 - bloodhound-python]] — collector (Linux/remote)
- [[05 - Vulnerability Analysis/11 - Certipy Find]] — ADCS misconfig discovery
- [[05 - Vulnerability Analysis/12 - BloodHound Queries]] — saved Cypher queries for attack paths

### Pre-auth Windows CVEs

- [[06 - Gaining Access/23 - ZeroLogon Reference]] — CVE-2020-1472 — DC machine account null
- [[06 - Gaining Access/24 - PrintNightmare Reference]] — CVE-2021-34527 — Print Spooler RCE
- [[06 - Gaining Access/25 - NoPac sAMAccountName]] — CVE-2021-42278/42287 — domain takeover
- [[06 - Gaining Access/21 - Certipy ESC Attacks]] — ADCS template abuse (ESC1-ESC11)
- [[06 - Gaining Access/10 - Searchsploit to Working Exploit]] — turn a CVE into something that runs

### Web/IIS path

- [[09 - Service Cheatsheets/06 - HTTP HTTPS]] — Windows IIS fingerprints
- [[04 - Enumeration/24 - Gobuster]] / [[04 - Enumeration/25 - ffuf]] / [[04 - Enumeration/26 - Feroxbuster]] — directory brute force
- [[06 - Gaining Access/37 - File Upload Bypass]] — ASPX upload into webroot
- [[11 - Shells Transfer Hashes/10 - ASPX Web Shells]] — webshell payloads
- [[06 - Gaining Access/39 - Tomcat Manager Exploit Chain]] — Windows Tomcat → WAR upload

### Brute force (last resort)

- [[06 - Gaining Access/01 - THC Hydra]] — RDP, SMB, WinRM brute force
- [[06 - Gaining Access/02 - Medusa]] — alternative bruteforcer
- [[06 - Gaining Access/03 - Ncrack]] — purpose-built for RDP

### Phishing / social engineering

- [[06 - Gaining Access/40 - SET Toolkit]] — credential harvester / website cloner
- [[06 - Gaining Access/41 - Gophish]] — campaign tracking
- [[06 - Gaining Access/42 - Evilginx2 AiTM]] — MFA session-cookie capture

## Phase 4 — Stabilize Shell / Get a Decent Session

- [[11 - Shells Transfer Hashes/02 - Windows Reverse Shells]] — PowerShell, nc.exe, MSF payloads
- [[11 - Shells Transfer Hashes/07 - Stabilizing PowerShell Shells]] — PowerShell shell upgrade ritual
- [[11 - Shells Transfer Hashes/05 - pwncat-cs Listener]] — auto-stabilizing listener
- [[06 - Gaining Access/09 - msfvenom Payload Cookbook]] — Windows payloads (exe, dll, hta, msi, lnk)
- [[06 - Gaining Access/08 - Metasploit Framework Workflow]] — meterpreter session handling

## Phase 5 — Privilege Escalation (local Windows)

### Enumeration (find the path)

- [[07 - Post-Exploitation/Windows/16 - Windows Manual Enumeration]] — whoami /priv, systeminfo, net, schtasks
- [[07 - Post-Exploitation/Windows/17 - WinPEAS]] — automated checks
- [[07 - Post-Exploitation/Windows/18 - PowerUp]] — privesc-focused PowerShell
- [[07 - Post-Exploitation/Windows/19 - Seatbelt]] — situational awareness (.NET)
- [[07 - Post-Exploitation/Windows/20 - PrivescCheck]] — alternative automated checker

### Service / config misconfigs

- [[07 - Post-Exploitation/Windows/21 - Unquoted Service Paths]] — classic intermediate-path drop
- [[07 - Post-Exploitation/Windows/22 - Weak Service ACLs]] — modify a service to run your binary
- [[07 - Post-Exploitation/Windows/23 - AlwaysInstallElevated]] — MSI installer as SYSTEM
- [[07 - Post-Exploitation/Windows/24 - Stored Credentials cmdkey]] — runas /savecred reuse
- [[07 - Post-Exploitation/Windows/28 - DLL Hijacking]] — writable-path DLL search-order abuse

### Token / impersonation

- [[07 - Post-Exploitation/Windows/25 - Token Impersonation]] — SeImpersonate / SeAssignPrimaryToken
- [[07 - Post-Exploitation/Windows/26 - Potato Attacks Family]] — JuicyPotato/PrintSpoofer/GodPotato
- [[07 - Post-Exploitation/Windows/27 - UAC Bypass (UACME)]] — medium → high integrity

## Phase 6 — Credential Harvest (after local admin / SYSTEM)

- [[07 - Post-Exploitation/Windows/29 - Mimikatz Deep Dive]] — LSASS, SAM, DCSync, ticket dump
- [[07 - Post-Exploitation/Windows/30 - Rubeus Deep Dive]] — ticket request, kerberoast, ASREP
- [[07 - Post-Exploitation/Windows/31 - SharpDPAPI]] — DPAPI-protected secrets (browser creds, RDP, vault)
- [[07 - Post-Exploitation/Windows/32 - LaZagne]] — broad credential sweeper
- [[07 - Post-Exploitation/Windows/33 - SAM SYSTEM NTDS Offline]] — offline hive parsing → hashes
- [[06 - Gaining Access/14 - secretsdump]] — remote SAM/LSA/NTDS dump via impacket

## Phase 7 — Lateral Movement

- [[07 - Post-Exploitation/Windows/34 - psexec wmiexec smbexec atexec]] — impacket execution methods
- [[07 - Post-Exploitation/Windows/35 - evil-winrm Lateral]] — PowerShell remoting with cred/hash
- [[07 - Post-Exploitation/Windows/36 - xfreerdp Pass-the-Hash]] — RDP with NTLM hash (Restricted Admin)
- [[07 - Post-Exploitation/Windows/37 - PowerShell Remoting]] — Enter-PSSession / Invoke-Command
- [[06 - Gaining Access/17 - Pass-the-Hash]] — reuse NTLM hash without cracking
- [[06 - Gaining Access/18 - Pass-the-Ticket]] — reuse Kerberos ticket
- [[06 - Gaining Access/19 - Overpass-the-Hash]] — NTLM → Kerberos TGT
- [[06 - Gaining Access/20 - DCSync]] — replicate the krbtgt + every user hash

### Pivoting / tunneling

- [[07 - Post-Exploitation/Windows/38 - chisel]] — HTTP-tunneled SOCKS
- [[07 - Post-Exploitation/Windows/39 - ligolo-ng]] — TUN-based pivot
- [[07 - Post-Exploitation/Windows/40 - sshuttle]] — VPN-over-SSH pivot
- [[07 - Post-Exploitation/Windows/41 - SSH Tunneling]] — `-L`, `-R`, `-D` patterns
- [[07 - Post-Exploitation/Windows/42 - proxychains]] — route tools through your pivot
- [[07 - Post-Exploitation/Windows/43 - plink Windows Pivot]] — putty CLI tunneling from Windows

## Phase 8 — Persistence / Domain Dominance

- [[07 - Post-Exploitation/Windows/45 - Windows Persistence]] — startup, registry, services, scheduled tasks
- [[07 - Post-Exploitation/Windows/46 - Golden Ticket]] — forge any TGT with krbtgt hash
- [[07 - Post-Exploitation/Windows/47 - Silver Ticket]] — forge service TGS with service hash
- [[07 - Post-Exploitation/Windows/48 - Skeleton Key]] — master password on the DC
- [[07 - Post-Exploitation/Windows/49 - DCSync Rights as Persistence]] — grant a user repl rights
- [[07 - Post-Exploitation/Windows/50 - GPO Abuse]] — push payloads via Group Policy
- [[07 - Post-Exploitation/Windows/51 - AdminSDHolder]] — stealth privileged-group persistence

## Phase 9 — Exfiltration

- [[07 - Post-Exploitation/Windows/52 - DNS Tunneling (dnscat2 iodine)]] — egress through DNS
- [[07 - Post-Exploitation/Windows/53 - ICMP Tunneling]] — egress through ICMP
- [[07 - Post-Exploitation/Windows/54 - rclone Cloud Exfil]] — push to S3/GCS/OneDrive
- [[07 - Post-Exploitation/Windows/55 - Compression and Encryption Pre-Exfil]] — 7z encryption before send
- [[11 - Shells Transfer Hashes/14 - PowerShell Download Methods]] — staging via DownloadString
- [[11 - Shells Transfer Hashes/15 - certutil Download (LOLBin)]] — LOLBin file transfer
- [[11 - Shells Transfer Hashes/16 - bitsadmin Download (LOLBin)]] — BITS-based download

## Phase 10 — Cover Tracks (study-only)

- [[08 - Tracks and Reporting/02 - Windows Event Log Clearing Study]] — wevtutil cl, Mimikatz event::clear
- [[08 - Tracks and Reporting/03 - Invoke-Phant0m]] — kill event log threads (academic)
- [[08 - Tracks and Reporting/04 - Timestomp Techniques]] — MACE manipulation
- [[08 - Tracks and Reporting/05 - Detection Hooks (Purple Team)]] — what SOCs catch you with

## Phase 11 — Reporting

- [[08 - Tracks and Reporting/06 - Report Structure]] — sections, audience layering
- [[08 - Tracks and Reporting/07 - Executive Summary Writing]] — the one page leadership reads
- [[08 - Tracks and Reporting/08 - Finding Template]] — copy-paste finding skeleton
- [[08 - Tracks and Reporting/09 - CVSS 3.1 and 4.0 Scoring]] — score every finding
- [[08 - Tracks and Reporting/10 - Attack Narrative]] — chronological compromise story
- [[08 - Tracks and Reporting/11 - Evidence Handling]] — chain of custody
- [[08 - Tracks and Reporting/13 - Post-Engagement Checklist]] — don't forget to clean up

## Cross-cutting references

- [[12 - HTB Workflows/09 - Windows AD Pattern]] — full HTB walkthrough pattern for AD boxes
- [[12 - HTB Workflows/08 - Windows Easy Pattern]] — non-AD Windows boxes
- [[12 - HTB Workflows/13 - Rabbit Hole Detector]] — when to back out
- [[20 - Hashcat Mode IDs]] — NTLM (1000), NetNTLMv2 (5600), Kerberos (13100/18200)
