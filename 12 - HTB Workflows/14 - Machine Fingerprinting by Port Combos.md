---
tags: [pentest, htb, fingerprinting, recon, methodology, both]
type: workflow
---
# Machine Fingerprinting by Port Combos

Before you run a single exploit, look at the port combination and decide **what kind of machine you are looking at**. The role determines priority and attack path. Not all hosts are equal — a Domain Controller is always top priority.

[[00 - README|Folder index]] · [[01 - When You See X Do Y|X→Y cheat card]] · [[02 - General Methodology|Methodology]]

> [!tip] Workflow
> 1. Run `nmap --open -p- --min-rate 5000 -Pn $RANGE -oA fullscan`
> 2. For each host, look up its open-port set in the table below.
> 3. Triage by role priority, then jump to the linked attack notes.

## Role → Port Signature Map

Ports marked `*` may or may not be present depending on configuration.

### Domain Controller (highest priority — always)

```text
53, 88, 135, 139, 389, 445, 464, 636, 3268, 3269, 3389*
```

**Tell:** 88 + 389 + 445 + 53 together. Add 3268 → it's a Global Catalog (almost always true of DCs).
**Attack path:** Kerberoast (need 1 valid cred), AS-REP roast (no cred needed), BloodHound for graph, PetitPotam / PrintSpooler coercion for machine-account NTLM relay, DCSync if you have replication rights.
**Owning a DC = owning the domain.**

→ [[26 - Kerberos and AD Services (88 464 3268 3269)|Kerberos cheatsheet]] · [[../06 - Gaining Access/15 - Kerberoasting]] · [[../06 - Gaining Access/16 - AS-REP Roasting]] · [[../06 - Gaining Access/20 - DCSync]]

### Read-Only Domain Controller (RODC)

```text
53, 88, 135, 139, 389, 445, 464, 636
```

**Tell:** Same as DC but typically no 3268/3269 (RODCs don't usually serve GC). Confirm by checking `msDS-RevealedList` on the computer object — that lists which user passwords are cached on this RODC.
**Attack path:** Lower impact than full DC. Cached creds for any users that logged in are crackable. If a Domain Admin ever authenticated to it (bad practice but common in branch offices), you may dump their hash.
**Reality check:** RODCs are surprisingly under-defended because admins assume "read-only = safe."

### Windows Workstation / Endpoint

```text
135, 139, 445, 3389*, 5985*
```

**Tell:** Just SMB + maybe RDP/WinRM. No 88/389/53/636.
**Attack path:** Primary lateral movement target. Check SMB signing (`nmap --script smb2-security-mode`) — disabled = NTLM relay is free. Local admin password reuse across workstations is the #1 finding on internal pentests.

→ [[11 - SMB (445 139)|SMB cheatsheet]] · [[../06 - Gaining Access/11 - Responder LLMNR NBT-NS Poisoning]] · [[../06 - Gaining Access/12 - ntlmrelayx]]

### Windows Member Server (generic)

```text
135, 139, 445, 3389*, 5985*, + role-specific ports
```

**Tell:** Workstation port set plus whatever role ports exist (1433 for SQL, 80/443 for IIS, etc.). WinRM (5985) more often enabled than on workstations.
**Attack path:** Identify the role from the extra ports — that becomes your primary vector. SMB is always the backup pivot path.

### Web Server (IIS / Apache / Nginx)

```text
80, 443, 8080*, 8443*
```

**Tell:** Web ports, often nothing else externally. Backend DB usually behind it.
**Attack path:** Fingerprint with WhatWeb / Wappalyzer. Directory brute force first 5 minutes (`feroxbuster`, `gobuster`). Look for: exposed `.git/`, `.env`, backup files (`.bak`, `.old`, `~`), admin panels, default creds. IIS often has WebDAV enabled — check with `davtest`. SSTI, file upload, outdated CMS are the standing top hits.

→ [[06 - HTTP HTTPS|HTTP cheatsheet]] · [[../06 - Gaining Access/37 - File Upload Bypass]] · [[../06 - Gaining Access/38 - LFI to RCE Patterns]]

### Mail Server (Exchange / Postfix)

```text
25, 110, 143, 443*, 465, 587, 993, 995
```

**Tell:** Many mail ports clustered. **443 + 25 + 587** strongly suggests Exchange (OWA + SMTP submission).
**Attack path:**

- **Exchange specifically:** ProxyLogon (CVE-2021-26855), ProxyShell (CVE-2021-34473), ProxyNotShell (CVE-2022-41040) — critical pre-auth RCE chains. **Always check Exchange build number.**
- OWA on 443 = password spraying surface. Lockout policies on OWA are routinely lax.
- Port 25 open relay → phish from a trusted IP.
- Autodiscover endpoint leaks internal hostnames.

```bash
# Exchange version fingerprint
curl -sk https://$IP/owa/auth/logon.aspx | grep -oP 'version=\K[\d.]+'
# Or via /ecp/Current/exporttool/microsoft.exchange.ediscovery.exporttool.application
```

→ [[04 - SMTP (25 465 587)|SMTP cheatsheet]] · [[../06 - Gaining Access/05 - Password Spraying Strategy]]

### Database Server (rarely external; usually internal pivot find)

```text
1433 (MSSQL), 3306 (MySQL), 5432 (PostgreSQL), 1521 (Oracle), 27017 (MongoDB)
```

**Tell:** A single DB port; check what authenticates to it (the web/app server you also found).
**Attack path:** Default creds first (`sa:`, `postgres:postgres`, `root:`, `scott:tiger`). Then file read/write privs (LOAD DATA INFILE, COPY FROM PROGRAM, xp_cmdshell). Linked servers in MSSQL chain cross-DB. MongoDB / Redis frequently no-auth.

→ [[13 - MSSQL (1433)]] · [[16 - MySQL MariaDB (3306)]] · [[18 - PostgreSQL (5432)]] · [[14 - Oracle TNS (1521)]] · [[23 - MongoDB (27017)]]

### Linux Server (general)

```text
22, 80*, 443*, 111*, 2049*, 8080*
```

**Tell:** SSH + maybe web + maybe NFS. No SMB, no Kerberos.
**Attack path:** SSH is the entry — brute force only with a credential pattern, otherwise look for stolen keys in OSINT. **111 + 2049 together = NFS** — enumerate exports immediately (`showmount -e`); `no_root_squash` is devastating. After shell: cron, SUID, capabilities.

→ [[02 - SSH (22)]] · [[15 - NFS (2049)]] · [[../07 - Post-Exploitation/Linux/00 - README]]

### Network Device (Router / Switch / Firewall)

```text
22, 23*, 80*, 161, 162, 443*
```

**Tell:** SSH/Telnet + SNMP + web management. No SMB, no DB, no app ports.
**Attack path:** SNMP is the intelligence source — `snmpwalk` with `public`/`private` yields routing tables, ARP caches, interface configs, sometimes the running config. Default web UI creds (admin/admin, cisco/cisco, root/admin) frequently work. Check the **exact firmware version** for CVE soup. Compromising a network device gives traffic visibility and routing control — high impact even without further pivot.

→ [[09 - SNMP (161)]] · [[../04 - Enumeration/16 - snmpwalk]]

### VPN / Remote Access Gateway

```text
443, 500, 1194*, 4500, 8443*
```

**Tell:** Internet-facing 443 paired with 500/4500 UDP. Banner / response on 443 identifies the product (Pulse, Fortinet, Cisco AnyConnect, GlobalProtect, SonicWall).
**Attack path:** **CVE lookup first** — VPN appliances are perennial pre-auth RCE / auth bypass targets. Pulse CVE-2019-11510, Fortinet CVE-2018-13379 & CVE-2022-42475, Citrix CVE-2019-19781, PAN-OS CVE-2019-1579. If no CVE hits, credential stuffing against the login portal is effective because users reuse VPN passwords with everything.

→ [[27 - IKE IPSec VPN (500 4500)|IKE cheatsheet]]

### Print Server / Printer

```text
9100, 515, 631, 80*, 443*, 161*
```

**Tell:** 9100 is the giveaway. SNMP + web UI almost always alongside.
**Attack path:** Web admin with default creds → export config → harvest LDAP/SMB/SMTP/AD credentials stored in config. PrintNightmare (CVE-2021-34527) is a **Windows Print Spooler** vuln, not a printer-firmware vuln — different target. Printer creds frequently work as service accounts elsewhere.

→ [[28 - Printers (9100 631 515)|Printer cheatsheet]]

### Jenkins / CI-CD Server

```text
8080, 8443*
```

**Tell:** 8080 with a `/` that returns "Jenkins" or `X-Jenkins:` header. Sometimes 50000 for agent JNLP.
**Attack path:** Default `admin:admin` on older installs. Anonymous Read enabled on many setups → browse jobs & build configs for secrets. **Script Console** (`/script`) executes arbitrary Groovy → instant RCE on the controller. Build credentials, deploy keys, and signing keys all live here. Compromise injects code into every project Jenkins builds.

```bash
curl -s http://$IP:8080/login | grep -i jenkins
curl -s http://$IP:8080/script    # 403 if authed, 200/redirect if anon
```

### Container Orchestration (Docker / Kubernetes)

```text
2375 / 2376 (Docker), 6443 / 8443 (K8s API), 10250 / 10255 (Kubelet), 2379 (etcd)
```

**Tell:** Any of these on a server you didn't expect them on = misconfigured exposure.
**Attack path:** 2375 with no TLS = mount host fs in a container = instant root. Kubelet 10250 with anonymous auth = exec into any pod on that node. K8s API on 6443 — try `system:anonymous` first. Etcd 2379 holds every Secret in cleartext if encryption-at-rest is not configured.

→ [[29 - Container APIs (Docker Kubernetes)|Container API cheatsheet]]

## Quick-recognition heuristics

| Port combo you see... | Says... |
| --- | --- |
| 88 + 389 + 445 + 53 | **Domain Controller** — drop everything else |
| 88 + 389 + 445 (no 3268) | **RODC** — check cached creds |
| 135 + 139 + 445 (no 88) | **Domain-joined Windows** (workstation or member server) |
| 445 alone with no 135 | Likely a Linux Samba box, not Windows |
| 22 + 111 + 2049 | **Linux + NFS** — `showmount -e` immediately |
| 25 + 587 + 993 + 443 | **Exchange** — check for ProxyLogon/ProxyShell builds |
| 161 + 9100 | **Printer / MFP** — config export → credential harvest |
| 500/udp + 4500/udp + 443 | **VPN appliance** — CVE lookup first |
| 2375 (open, no TLS) | **Docker daemon exposed** — game over |
| 6443 with anonymous 200 to /version | **K8s API** — probe RBAC |

## Mindset reminders

- Ports are surface area, not guaranteed vulns. Verify with `-sV` and pin the exact version.
- **Combinations matter more than individual ports.**
- Check NTLM relay opportunity before anything else on Windows networks. No SMB signing = free lateral movement.
- Every database port = ask: default creds? File read/write? OS commands?
- Legacy ports (21, 23, 111, 161) = legacy security mindset. Default creds and cleartext everywhere.
- Document every open port and service version. That context is your roadmap for the rest of the engagement.

## Related

- [[01 - When You See X Do Y]] — service-level pattern recognition (per-vuln, not per-machine)
- [[02 - General Methodology]] — overall flow (now includes Port-Based Triage Tier 1/2/3)
- [[../09 - Service Cheatsheets/00 - README]] — per-port attack notes
