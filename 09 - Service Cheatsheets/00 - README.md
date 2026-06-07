---
tags: [pentest, cheatsheet, services, moc, both]
type: reference
---
# 09 - Service Cheatsheets

Per-protocol attack flow: enumerate → authenticate → abuse.

[[00 - Vault Index|Home]]

## Files in this folder

| Port(s) | Service | File |
| ---: | --- | --- |
| 21 | FTP | [[01 - FTP (21)]] |
| 22 | SSH | [[02 - SSH (22)]] |
| 23 | Telnet | [[03 - Telnet (23)]] |
| 25 / 465 / 587 | SMTP | [[04 - SMTP (25 465 587)]] |
| 53 | DNS | [[05 - DNS (53)]] |
| 80 / 443 / 8080 / 8443 | HTTP / HTTPS | [[06 - HTTP HTTPS]] |
| 110 | POP3 | [[07 - POP3 (110)]] |
| 143 / 993 | IMAP / IMAPS | [[08 - IMAP (143 993)]] |
| 161 (udp) | SNMP | [[09 - SNMP (161)]] |
| 389 / 636 | LDAP / LDAPS | [[10 - LDAP LDAPS (389 636)]] |
| 445 / 139 | SMB / NetBIOS | [[11 - SMB (445 139)]] |
| 873 | rsync | [[12 - rsync (873)]] |
| 1433 | MSSQL | [[13 - MSSQL (1433)]] |
| 1521 | Oracle TNS | [[14 - Oracle TNS (1521)]] |
| 2049 | NFS | [[15 - NFS (2049)]] |
| 3306 | MySQL / MariaDB | [[16 - MySQL MariaDB (3306)]] |
| 3389 | RDP | [[17 - RDP (3389)]] |
| 5432 | PostgreSQL | [[18 - PostgreSQL (5432)]] |
| 5900-5901 | VNC | [[19 - VNC (5900)]] |
| 5985 / 5986 | WinRM | [[20 - WinRM (5985 5986)]] |
| 6379 | Redis | [[21 - Redis (6379)]] |
| 11211 | Memcached | [[22 - Memcached (11211)]] |
| 27017 | MongoDB | [[23 - MongoDB (27017)]] |
| — | Cloud Storage | [[24 - Cloud Storage S3 GCS Azure]] |
| 623 (udp) | IPMI / BMC | [[25 - IPMI (623)]] |
| 88 / 464 / 3268 / 3269 | Kerberos & AD GC | [[26 - Kerberos and AD Services (88 464 3268 3269)]] |
| 500 / 4500 (udp) | IKE / IPSec VPN | [[27 - IKE IPSec VPN (500 4500)]] |
| 9100 / 631 / 515 | Printers (JetDirect/IPP/LPD) | [[28 - Printers (9100 631 515)]] |
| 2375 / 6443 / 10250 | Docker / Kubernetes APIs | [[29 - Container APIs (Docker Kubernetes)]] |

> [!tip] When you don't recognize a port
> `nmap -sV -p<port> --version-intensity 9 <ip>` for version, then `searchsploit <product> <version>`.

> [!tip] Triage order
> Don't attack left-to-right. See [[../12 - HTB Workflows/02 - General Methodology#Port-Based Triage (Tier 1 / 2 / 3)|Port-Based Triage tiers]] and [[../12 - HTB Workflows/14 - Machine Fingerprinting by Port Combos|Machine Fingerprinting by Port Combos]] before picking a target.
