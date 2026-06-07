---
tags: [moc, guide, usage]
type: usage-guide
---

# 00 — USE ME

**Open me at the start of every box.** This is a navigation guide, not a tutorial — scan, jump, close.

[[00 - Vault Index|Index]] · [[Windows Attack Map]] · [[Linux Attack Map]] · [[12 - HTB Workflows/01 - When You See X Do Y|X→Y card]] · [[12 - HTB Workflows/02 - General Methodology|Methodology]]

---

## When to use what

| Situation | Open this |
| --- | --- |
| Don't know what to do next on a box | [[12 - HTB Workflows/01 - When You See X Do Y]] |
| Want the universal flow | [[12 - HTB Workflows/02 - General Methodology]] |
| Looking at port scan, don't know what kind of machine it is | [[12 - HTB Workflows/14 - Machine Fingerprinting by Port Combos]] |
| Need attack ideas for a specific port | [[09 - Service Cheatsheets/00 - README]] |
| Need a tool I haven't used in a month | Tag search (see below) |
| Doing a Windows box | [[Windows Attack Map]] |
| Doing a Linux box | [[Linux Attack Map]] |
| Need a payload (reverse shell, msfvenom, etc.) | Folder [[11 - Shells Transfer Hashes/00 - README\|11 - Shells Transfer Hashes]] |
| Need a hash mode for cracking | [[20 - Hashcat Mode IDs]] |
| Stuck > 2 hours | [[12 - HTB Workflows/13 - Rabbit Hole Detector]] then [[12 - HTB Workflows/12 - Common Pitfalls and Time Sinks]] |
| Need to write the report | Folder [[08 - Tracks and Reporting/00 - README\|08 - Tracks and Reporting]] |

## MOC vs. tag search vs. folder navigation

Three ways to find a note. Pick the right one.

| Method | When | Cost |
| --- | --- | --- |
| **MOC** (Attack Map, this file, folder READMEs) | You know roughly *what phase* you're in but not *which note*. Best for "I have a foothold, what now?" | Free — already curated |
| **Tag search** (`#privesc`, `#active-directory`, etc.) | You want *every* note touching a concept across folders | Quick, narrows to ~5-30 hits |
| **Folder navigation** | You already know the tool name or service | Fastest when you know the exact target |
| **Ctrl+O (Quick Switcher)** | You know the file name | Fastest of all when you remember it |

## Obsidian tags reference

Every note carries a phase / OS tag. Open the **tag pane** (Ctrl+Shift+# or sidebar) to filter.

| Tag | What it means |
| --- | --- |
| `#windows` | Specific to Windows targets |
| `#linux` | Specific to Linux targets |
| `#both` | Cross-OS (methodology, port refs, cracking) |
| `#active-directory` | AD-specific (DC attacks, Kerberos, LDAP) |
| `#web` | Web application attack surface |
| `#recon` | Phase 1-2 — discovery and enumeration |
| `#initial-access` | Phase 3 — first foothold / shell |
| `#privesc` | Local privilege escalation |
| `#lateral-movement` | Pivoting / cross-host movement / persistence |
| `#exfil` | Exfiltration channels and packaging |

**Useful combined queries (Obsidian search bar):**

```text
tag:#active-directory tag:#privesc
tag:#linux tag:#initial-access -tag:#web
tag:#windows tag:#lateral-movement
```

## Recommended workflow (box start → root flag)

```text
1.  Open [[12 - HTB Workflows/04 - Note-Taking Template]] → copy to a new note
    named after the box. ALL notes for this box go in that file.

2.  Run the nmap from [[12 - HTB Workflows/03 - Pre-Flight Checklist]].

3.  Take the port list → cross-reference with
    [[12 - HTB Workflows/14 - Machine Fingerprinting by Port Combos]]
    → decide what kind of machine you're looking at.

4.  Open the right Attack Map:
    [[Windows Attack Map]] OR [[Linux Attack Map]]
    → jump to Phase 2 (Service Enumeration).

5.  For each open port, open its file in [[09 - Service Cheatsheets/00 - README]].
    Run the Enumerate section verbatim. Save output to your box note.

6.  When stuck on a service → [[12 - HTB Workflows/01 - When You See X Do Y]]
    to find the next move from a symptom.

7.  Got a shell? → Stabilize via [[11 - Shells Transfer Hashes/04 - PTY Upgrade Ritual]]
    → then open Phase 5 of the Attack Map for privesc enum.

8.  Privesc working? → Phase 6 of Attack Map for cred harvest.

9.  Got root? → Phase 7-9 of Attack Map for pivot, persistence, exfil
    (or just the flag on HTB).

10. WRITE THE NOTES BEFORE STARTING THE NEXT BOX.
    Use [[12 - HTB Workflows/04 - Note-Taking Template]] sections.
```

## Quick-reference decision tree — "what do I open first"

### Based on what nmap returned

```text
Port 445 OR 139 open?
├── Plus 88 + 389 + 53?     → DOMAIN CONTROLLER
│                              [[Windows Attack Map]] § Phase 3 → AD-specific
│                              Start with [[06 - Gaining Access/16 - AS-REP Roasting]]
│
├── Plus 135 only?          → Windows workstation / server
│                              [[Windows Attack Map]] § Phase 2
│                              [[09 - Service Cheatsheets/11 - SMB (445 139)]] — check signing
│
└── Standalone (no 135)?    → Probably Linux Samba
                               [[09 - Service Cheatsheets/11 - SMB (445 139)]]

Port 22 open?
├── Plus 80/443?            → Linux web server
│                              [[Linux Attack Map]] § Phase 3 — Web path
│
├── Plus 111 + 2049?        → Linux + NFS
│                              [[09 - Service Cheatsheets/15 - NFS (2049)]] immediately
│
└── 22 alone?               → SSH-only host
                               Banner grab → CVE check → only brute force last

Port 80 / 443 / 8080 / 8443 open and nothing else interesting?
└── Web-heavy box           → [[12 - HTB Workflows/10 - Web-Heavy Box Pattern]]
                               Dir brute force in parallel: feroxbuster
                               Tech fingerprint: WhatWeb / Wappalyzer / Nuclei

Port 2375 (Docker) or 6443 (K8s API)?
└── INSTANT PRIORITY        → [[09 - Service Cheatsheets/29 - Container APIs (Docker Kubernetes)]]
                               Try anonymous access first

Port 623 UDP?
└── IPMI / BMC              → [[09 - Service Cheatsheets/25 - IPMI (623)]]
                               Try default vendor creds (root/calvin etc.) AND RAKP hash dump

Port 9100?
└── Printer / MFP           → [[09 - Service Cheatsheets/28 - Printers (9100 631 515)]]
                               Web admin default creds → config export → harvest LDAP/SMB creds

Port 500 / 4500 UDP?
└── VPN appliance           → [[09 - Service Cheatsheets/27 - IKE IPSec VPN (500 4500)]]
                               CVE lookup on product first; aggressive-mode PSK second

Port 1433 / 3306 / 5432 / 27017?
└── Database exposed        → Per-service cheatsheet
                               [[09 - Service Cheatsheets/00 - README]] for the right one
                               Default creds → file r/w → RCE

Port 6379 with no auth banner?
└── Redis pwn               → [[09 - Service Cheatsheets/21 - Redis (6379)]]
                               SSH key inject or webroot shell drop = instant RCE
```

### After you have a shell

```text
Linux shell?
├── Always: id, sudo -l, find SUID, getcap -r /, crontab -l, /etc/cron*
├── Then:   LinPEAS in background  → [[07 - Post-Exploitation/Linux/02 - LinPEAS]]
└── Stuck:  pspy + 5 min observation → [[07 - Post-Exploitation/Linux/05 - pspy]]

Windows shell?
├── Always: whoami /priv, whoami /groups, systeminfo, net user, net localgroup
├── Then:   WinPEAS in background  → [[07 - Post-Exploitation/Windows/17 - WinPEAS]]
├── SeImpersonate present?         → Potato family
│                                    [[07 - Post-Exploitation/Windows/26 - Potato Attacks Family]]
└── Stuck:  PowerUp + Seatbelt     → [[07 - Post-Exploitation/Windows/18 - PowerUp]]
```

### After you find creds

```text
Cred = NTLM hash for user?
├── Try Pass-the-Hash with NetExec    → [[04 - Enumeration/05 - NetExec (nxc)]]
├── Spray same hash across subnet     → [[06 - Gaining Access/17 - Pass-the-Hash]]
└── If admin: secretsdump the box     → [[06 - Gaining Access/14 - secretsdump]]

Cred = Kerberos ticket?
└── [[06 - Gaining Access/18 - Pass-the-Ticket]]

Cred = cleartext password?
├── Try SSH everywhere                → key + password reuse is rampant
├── Try SMB / WinRM / RDP             → [[04 - Enumeration/05 - NetExec (nxc)]]
├── Try LDAP bind                     → [[04 - Enumeration/07 - ldapsearch]]
└── BloodHound from this user         → [[04 - Enumeration/09 - BloodHound]]

Cred = hash to crack?
├── Identify the hash                 → [[11 - Shells Transfer Hashes/26 - Hash Identification]]
├── Pick the right mode               → [[11 - Shells Transfer Hashes/20 - Hashcat Mode IDs]]
└── Run                               → [[06 - Gaining Access/06 - Hashcat Core]]
```

## Time-boxing (don't ignore this)

| Phase | Easy box | Hard box | Bail when... |
| --- | --- | --- | --- |
| Recon + enum | 30 min | 60-90 min | You've scanned 3x, still nothing → check UDP / vhosts / IPv6 |
| Foothold | 30 min | 60-120 min | 3+ leads tried, none work → re-enumerate |
| Privesc | 30 min | 60-90 min | LinPEAS/WinPEAS yellow + red checked, no win → re-enumerate after a break |
| **Hard stop** | **2 hours stuck** | **3-4 hours stuck** | Walk away, write what you've tried, come back fresh |

If you blow these limits, you missed something in enumeration — not the exploit.

## Vault hygiene

- **Per-box notes** go in your own folder outside the vault, or use the template in [[12 - HTB Workflows/04 - Note-Taking Template]].
- **Don't edit cheatsheets mid-box** — add observations to the per-box note. Update cheatsheets only after the box is done and you've learned something durable.
- **Add tags as you create new notes** so this MOC / search keeps working. Tag vocabulary is fixed (see table above).

## When the vault doesn't have what you need

In order:

1. `searchsploit <product> <version>` — local
2. https://gtfobins.github.io — Linux binary abuse
3. https://lolbas-project.github.io — Windows LOLBins
4. https://www.revshells.com — reverse shell generator
5. https://hashcat.net/wiki/doku.php?id=example_hashes — hash mode identification
6. https://book.hacktricks.wiki — broader reference (use sparingly; their methodology differs)

Then come back and add a note to the vault if you used something new.
