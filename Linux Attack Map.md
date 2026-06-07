---
tags: [moc, attack-map, linux]
type: map-of-content
---

# Linux Attack Map

Map of Content for **Linux** targets — bare-metal, VM, container host, Docker/K8s nodes. Organized by engagement phase. For Windows / AD targets, see [[Windows Attack Map]].

[[00 - Vault Index|Home]] · [[00 - USE ME|How to use this vault]] · [[12 - HTB Workflows/02 - General Methodology|Methodology]]

> [!tip] How to use this MOC
> Scan the phase headings, find the closest match to your current situation, jump to the note. Don't read top-to-bottom — Ctrl+F your symptom.

---

## Phase 1 — Recon

Is it Linux? What flavor and role?

- [[12 - HTB Workflows/14 - Machine Fingerprinting by Port Combos]] — Linux port signatures (SSH + NFS, web stacks, container hosts)
- [[12 - HTB Workflows/01 - When You See X Do Y]] — service → next step
- [[03 - Scanning/04 - nmap Host Discovery]] — host discovery (ARP/ICMP)
- [[03 - Scanning/01 - arp-scan]] / [[03 - Scanning/02 - netdiscover]] — L2 discovery on internal LAN
- [[03 - Scanning/05 - nmap Basics]] — full TCP + scripts
- [[03 - Scanning/07 - nmap Service and Version Detection]] — pin SSH/HTTP/Samba versions for CVE work
- [[02 - Reconnaissance and OSINT/15 - Shodan]] — external Linux assets, banner intel
- [[02 - Reconnaissance and OSINT/14 - Google Dorking]] — exposed configs, .env files, dashboards

## Phase 2 — Service Enumeration (no creds)

- [[09 - Service Cheatsheets/02 - SSH (22)]] — banner, version, key auth
- [[09 - Service Cheatsheets/06 - HTTP HTTPS]] — Apache / Nginx fingerprints
- [[09 - Service Cheatsheets/15 - NFS (2049)]] — `showmount -e`, no_root_squash
- [[09 - Service Cheatsheets/12 - rsync (873)]] — anonymous module enum
- [[09 - Service Cheatsheets/05 - DNS (53)]] — zone transfers on internal BIND
- [[09 - Service Cheatsheets/16 - MySQL MariaDB (3306)]] — common LAMP backend
- [[09 - Service Cheatsheets/18 - PostgreSQL (5432)]] — common backend
- [[09 - Service Cheatsheets/21 - Redis (6379)]] — frequently no-auth → RCE
- [[09 - Service Cheatsheets/22 - Memcached (11211)]] — cache dump, session tokens
- [[09 - Service Cheatsheets/23 - MongoDB (27017)]] — no-auth on older
- [[09 - Service Cheatsheets/29 - Container APIs (Docker Kubernetes)]] — Docker 2375, K8s 6443/10250
- [[04 - Enumeration/15 - onesixtyone]] / [[04 - Enumeration/16 - snmpwalk]] — SNMP on Linux servers
- [[04 - Enumeration/32 - NFS Enumeration]] — exports, mount, write tests
- [[04 - Enumeration/22 - smtp-user-enum]] — VRFY/EXPN on Postfix/Exim

## Phase 3 — Initial Access (foothold)

### SSH path

- [[09 - Service Cheatsheets/02 - SSH (22)]] — version-based CVEs (OpenSSH < 7.x username enum)
- [[06 - Gaining Access/01 - THC Hydra]] — brute force (last resort)
- Stolen `id_rsa` / passphrase crack via `ssh2john` → [[06 - Gaining Access/07 - John the Ripper Core]]

### Web path

- [[04 - Enumeration/24 - Gobuster]] / [[04 - Enumeration/25 - ffuf]] / [[04 - Enumeration/26 - Feroxbuster]] — dir brute force
- [[04 - Enumeration/27 - dirsearch]] — alt with smart wordlists
- [[04 - Enumeration/28 - WPScan]] — WordPress
- [[04 - Enumeration/29 - JoomScan]] — Joomla
- [[04 - Enumeration/30 - Droopescan]] — Drupal
- [[04 - Enumeration/31 - Nikto]] — quick web vuln sweep
- [[06 - Gaining Access/29 - SQLMap]] — SQL injection automation
- [[06 - Gaining Access/32 - Manual SQLi Payloads]] — when SQLMap can't crack the form
- [[06 - Gaining Access/30 - XSStrike]] — XSS automation
- [[06 - Gaining Access/31 - Commix]] — command injection automation
- [[06 - Gaining Access/33 - XSS Payloads]] — manual payloads
- [[06 - Gaining Access/34 - SSRF Cheats]] — SSRF → cloud metadata
- [[06 - Gaining Access/35 - XXE Payloads]] — XML external entity
- [[06 - Gaining Access/36 - Deserialization (ysoserial)]] — Java/PHP deserialization
- [[06 - Gaining Access/37 - File Upload Bypass]] — PHP/JSP into webroot
- [[06 - Gaining Access/38 - LFI to RCE Patterns]] — log poisoning, php filter, SSH key plant
- [[11 - Shells Transfer Hashes/08 - PHP Web Shells]] — PHP payloads
- [[11 - Shells Transfer Hashes/09 - JSP Web Shells (Tomcat)]] — JSP payloads
- [[11 - Shells Transfer Hashes/11 - Weevely Stealth PHP]] — encrypted PHP backdoor

### Database path (no auth or default creds)

- [[09 - Service Cheatsheets/21 - Redis (6379)]] — SSH key inject / webroot shell
- [[09 - Service Cheatsheets/18 - PostgreSQL (5432)]] — `COPY ... PROGRAM` for RCE
- [[09 - Service Cheatsheets/16 - MySQL MariaDB (3306)]] — UDF / INTO OUTFILE
- [[04 - Enumeration/34 - MySQL Enumeration]]
- [[04 - Enumeration/35 - PostgreSQL Enumeration]]
- [[04 - Enumeration/36 - Redis Enumeration]]
- [[04 - Enumeration/37 - MongoDB Enumeration]]

### NFS / file-share path

- [[09 - Service Cheatsheets/15 - NFS (2049)]] — mount + write as root w/ no_root_squash
- SSH key plant in `/root/.ssh/authorized_keys` (mount-and-write pattern)

### Container API path

- [[09 - Service Cheatsheets/29 - Container APIs (Docker Kubernetes)]] — Docker 2375 escape, K8s anonymous
- [[07 - Post-Exploitation/Linux/12 - Docker LXD Group Privesc]] — local docker-group escape

### Searchsploit-driven exploits

- [[05 - Vulnerability Analysis/01 - searchsploit]] — find a working CVE
- [[06 - Gaining Access/10 - Searchsploit to Working Exploit]] — turn it into something that runs

## Phase 4 — Stabilize Shell

- [[11 - Shells Transfer Hashes/01 - Linux Reverse Shells]] — bash, nc, python, php, perl one-liners
- [[11 - Shells Transfer Hashes/03 - Bind Shells]] — when you can't egress
- [[11 - Shells Transfer Hashes/04 - PTY Upgrade Ritual]] — the canonical `python -c pty.spawn` flow
- [[11 - Shells Transfer Hashes/05 - pwncat-cs Listener]] — auto-stabilizing listener
- [[11 - Shells Transfer Hashes/06 - socat Full TTY]] — full TTY in one connection

## Phase 5 — Privilege Escalation (local Linux)

### Enumerate

- [[07 - Post-Exploitation/Linux/01 - Linux Manual Enumeration]] — id, sudo -l, /etc, history, cron
- [[07 - Post-Exploitation/Linux/02 - LinPEAS]] — automated checks
- [[07 - Post-Exploitation/Linux/03 - LinEnum]] — older alternative
- [[07 - Post-Exploitation/Linux/04 - Linux Exploit Suggester]] — kernel CVE candidates
- [[07 - Post-Exploitation/Linux/05 - pspy]] — passive process monitor for cron/scripts
- [[07 - Post-Exploitation/Linux/06 - GTFOBins Reference]] — bookmark for every shell binary

### Sudo / SUID / capabilities

- [[07 - Post-Exploitation/Linux/07 - Sudo Privesc]] — sudo -l → GTFOBins
- [[07 - Post-Exploitation/Linux/08 - SUID SGID Privesc]] — `find / -perm -4000`
- [[07 - Post-Exploitation/Linux/09 - Capabilities Privesc]] — `getcap -r /`

### Cron / wildcards / services

- [[07 - Post-Exploitation/Linux/10 - Cron Privesc]] — writable cron scripts, wildcard injection
- [[07 - Post-Exploitation/Linux/11 - Wildcard Injection]] — tar/rsync wildcard tricks
- [[07 - Post-Exploitation/Linux/13 - PATH Hijacking]] — script calling unqualified binary

### Container / kernel

- [[07 - Post-Exploitation/Linux/12 - Docker LXD Group Privesc]] — `docker run -v /:/host`
- [[07 - Post-Exploitation/Linux/14 - Linux Kernel Exploits (PwnKit Dirty Pipe)]] — CVE-2021-4034, CVE-2022-0847
- [[07 - Post-Exploitation/Linux/15 - Writable etc passwd Trick]] — append a root user

## Phase 6 — Credential Harvest (after root)

- `/etc/shadow` → [[06 - Gaining Access/07 - John the Ripper Core]] mode `sha512crypt` / hashcat 1800
- `~/.ssh/` keys, `~/.bash_history`, `~/.aws/credentials`, `~/.kube/config`
- `/var/log/auth.log` for failed-login user lists
- `/var/www/html/` configs (wp-config.php, .env, database.yml)
- Memory dump via `gcore` / `process_vm_readv` for live secrets

## Phase 7 — Lateral Movement / Pivoting

- [[06 - Gaining Access/26 - evil-winrm]] / [[06 - Gaining Access/27 - psexec wmiexec smbexec]] — pivot to Windows from Linux
- [[06 - Gaining Access/28 - xfreerdp]] — RDP from Linux
- [[07 - Post-Exploitation/Windows/38 - chisel]] — SOCKS via HTTP from Linux
- [[07 - Post-Exploitation/Windows/39 - ligolo-ng]] — TUN pivot
- [[07 - Post-Exploitation/Windows/40 - sshuttle]] — VPN-over-SSH
- [[07 - Post-Exploitation/Windows/41 - SSH Tunneling]] — `-L`, `-R`, `-D`
- [[07 - Post-Exploitation/Windows/42 - proxychains]] — route any tool through your pivot
- SSH key reuse: try a recovered key against every other Linux host on the subnet

## Phase 8 — Persistence

- [[07 - Post-Exploitation/Windows/44 - Linux Persistence]] — cron, systemd unit, .bashrc, SSH authorized_keys, SUID binary
- Backdoor an init script with a delay before reverse shell
- Add a UID 0 user to /etc/passwd (loud — only when stealth isn't required)

## Phase 9 — Exfiltration

- [[07 - Post-Exploitation/Windows/52 - DNS Tunneling (dnscat2 iodine)]] — DNS egress
- [[07 - Post-Exploitation/Windows/53 - ICMP Tunneling]] — ICMP egress
- [[07 - Post-Exploitation/Windows/54 - rclone Cloud Exfil]] — S3/GCS/Azure/Drive
- [[07 - Post-Exploitation/Windows/55 - Compression and Encryption Pre-Exfil]] — 7z encrypted archive
- [[11 - Shells Transfer Hashes/12 - Python HTTP Server Transfer]] — quick stage server
- [[11 - Shells Transfer Hashes/13 - SMB Server Transfer (impacket)]] — when Windows needs the file
- [[11 - Shells Transfer Hashes/18 - scp and rsync]] — classic transfer
- [[11 - Shells Transfer Hashes/19 - FTP Client Commands]]

## Phase 10 — Cover Tracks (study-only)

- [[08 - Tracks and Reporting/01 - Linux Log Clearing Study]] — `/var/log` purge patterns, history wipe
- [[08 - Tracks and Reporting/04 - Timestomp Techniques]] — `touch -t`, `touch -r`
- [[08 - Tracks and Reporting/05 - Detection Hooks (Purple Team)]] — what defenders catch

## Phase 11 — Reporting

Same notes as Windows side:

- [[08 - Tracks and Reporting/06 - Report Structure]]
- [[08 - Tracks and Reporting/07 - Executive Summary Writing]]
- [[08 - Tracks and Reporting/08 - Finding Template]]
- [[08 - Tracks and Reporting/09 - CVSS 3.1 and 4.0 Scoring]]
- [[08 - Tracks and Reporting/10 - Attack Narrative]]
- [[08 - Tracks and Reporting/11 - Evidence Handling]]
- [[08 - Tracks and Reporting/13 - Post-Engagement Checklist]]

## Cross-cutting references

- [[12 - HTB Workflows/06 - Linux Easy Pattern]] — easy box walkthrough
- [[12 - HTB Workflows/07 - Linux Hard Pattern]] — hard box walkthrough
- [[12 - HTB Workflows/10 - Web-Heavy Box Pattern]] — most Linux HTB boxes are web-driven
- [[12 - HTB Workflows/13 - Rabbit Hole Detector]] — when to bail out
- [[20 - Hashcat Mode IDs]] — sha512crypt (1800), md5crypt (500), bcrypt (3200)
