---
tags: [pentest, htb, methodology, both]
type: workflow
---
# General Methodology

The universal flow for every HTB / lab / OSCP-style box.

[[00 - README|Folder index]] · [[01 - When You See X Do Y|X→Y cheat card]]

## The flow

```text
┌─────────────────────────────────────────────────────────────────┐
│ 1. RECON (5-10 min)                                             │
│    - Fast full TCP: nmap -p- --min-rate 5000 -Pn ip             │
│    - Then targeted: nmap -sC -sV -p<openports> -Pn ip -oA full  │
│    - UDP top 100 in parallel: sudo nmap -sU --top-ports 100     │
│    - Add hostname(s) to /etc/hosts                              │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ 2. ENUMERATE EVERY SERVICE FULLY (60-80% of total time)         │
│    For each open port, use [[../09 - Service Cheatsheets       │
│    /00 - README]] to find the per-protocol flow.                │
│    Do not move on until you've tried:                           │
│      - Anonymous / default access                               │
│      - Version → CVE / exploit-db lookup                        │
│      - Banner grab + manual interaction                         │
│      - Brute / fuzz where applicable                            │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ 3. FIND FOOTHOLD                                                │
│    - Public exploit?                                            │
│    - Default creds?                                             │
│    - Web vuln (SQLi, LFI, RCE, file upload)?                    │
│    - Anonymous SMB / FTP with writable path?                    │
│    - AD: AS-REP / Kerberoast / null session                     │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ 4. STABILIZE SHELL                                              │
│    PTY upgrade ritual: see [[../11 - Shells Transfer Hashes    │
│    /04 - PTY Upgrade Ritual]]                                   │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ 5. USER ENUM + USER FLAG                                        │
│    - id, whoami, hostname, uname -a                             │
│    - find / -name "user.txt" 2>/dev/null                        │
│    - .bash_history, /tmp, /home/*, configs in webroot           │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ 6. PRIV-ESC                                                     │
│    Linux:                                                       │
│      - sudo -l        ← #1 lead                                 │
│      - SUID/cap/cron  ← #2 lead                                 │
│      - LinPEAS / pspy ← exhaustive                              │
│    Windows:                                                     │
│      - whoami /priv   ← SeImpersonate? Go Potato                │
│      - WinPEAS / PowerUp                                        │
│      - cmdkey /list, runas /savecred                            │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ 7. ROOT FLAG + LOOT                                             │
│    - /root/root.txt                                             │
│    - C:\Users\Administrator\Desktop\root.txt                    │
│    - Then dump creds for the writeup                            │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ 8. WRITEUP (do this BEFORE moving to next box)                  │
│    - Commands run                                               │
│    - Screenshots of key wins                                    │
│    - What I'd do differently                                    │
│    - What I learned (technique, tool, gotcha)                   │
└─────────────────────────────────────────────────────────────────┘
```

## Port-Based Triage (Tier 1 / 2 / 3)

Once recon is done, don't attack ports left-to-right. Triage by what's likely to win fastest. Use this order:

### Tier 1 — Immediate high value (attack first)

| Port | Why first |
| ---: | --- |
| **445** (SMB) | Check signing → enumerate shares → attempt NTLM relay. Highest reward on Windows networks. |
| **88** (Kerberos) | You found a DC. Start Kerberoasting + AS-REP Roasting **immediately**, before anything else. |
| **389 / 3268** (LDAP / GC) | Try anonymous bind. If auth required, any valid cred + BloodHound. |
| **5985 / 5986** (WinRM) | If you have a cred or hash already, this is the remote-execution path. |
| **8080 / 8443** with Jenkins or Tomcat | Default creds → Script console / WAR upload = instant RCE. |
| **6379** (Redis, no auth) | SSH key injection or webroot shell drop for instant RCE. |
| **2375** (Docker API, no TLS) | Mount host filesystem in a container → root on host. |

### Tier 2 — Enumerate and probe

| Port | Why |
| ---: | --- |
| **135** (MS-RPC) | Enumerate registered endpoints, pivot via WMI / DCOM. |
| **53** (DNS) | Attempt zone transfer. Set up for DNS tunneling if needed. |
| **161** (SNMP) | Walk MIB with defaul