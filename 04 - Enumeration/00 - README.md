---
tags: [pentest, enumeration, moc, recon]
phase: 3
---
# 04 - Enumeration

Active interrogation of identified services - users, shares, banners, versions, AD objects.

[[00 - Vault Index|Home]] · Prev: [[03 - Scanning/00 - README|Scanning]] · Next: [[05 - Vulnerability Analysis/00 - README|Vuln Analysis]]

## Files in this folder

### SMB / NetBIOS
- [[01 - enum4linux and enum4linux-ng]]
- [[02 - smbclient]]
- [[03 - smbmap]]
- [[04 - rpcclient]]
- [[05 - NetExec (nxc)]]
- [[06 - nmap SMB Scripts]]

### LDAP / AD
- [[07 - ldapsearch]]
- [[08 - windapsearch]]
- [[09 - BloodHound]]
- [[10 - SharpHound]]
- [[11 - bloodhound-python]]

### Kerberos
- [[12 - kerbrute]]
- [[13 - Impacket GetNPUsers (AS-REP)]]
- [[14 - Impacket GetUserSPNs (Kerberoast)]]

### SNMP
- [[15 - onesixtyone]]
- [[16 - snmpwalk]]
- [[17 - snmp-check]]

### DNS
- [[18 - dnsenum]]
- [[19 - dnsrecon]]
- [[20 - fierce]]
- [[21 - dnsmap]]

### Mail
- [[22 - smtp-user-enum]]
- [[23 - swaks]]

### Web (directory + content discovery)
- [[24 - Gobuster]]
- [[25 - ffuf]]
- [[26 - Feroxbuster]]
- [[27 - dirsearch]]
- [[28 - WPScan]]
- [[29 - JoomScan]]
- [[30 - Droopescan]]
- [[31 - Nikto]]

### File services
- [[32 - NFS Enumeration]]

### Databases
- [[33 - MSSQL Enumeration]]
- [[34 - MySQL Enumeration]]
- [[35 - PostgreSQL Enumeration]]
- [[36 - Redis Enumeration]]
- [[37 - MongoDB Enumeration]]

## Triage checklist - what to enumerate based on what's open

| Port | Open → look at |
| --- | --- |
| 21 | [[09 - Service Cheatsheets/01 - FTP (21)\|FTP]] - anon login, vsftpd backdoor |
| 22 | [[09 - Service Cheatsheets/02 - SSH (22)\|SSH]] - banner, user enum (legacy), keys |
| 25 | [[22 - smtp-user-enum]], VRFY/EXPN |
| 53 | [[18 - dnsenum]], zone transfer |
| 80/443 | dir brute, vhosts, [[31 - Nikto]], [[28 - WPScan]] |
| 110/143 | [[09 - Service Cheatsheets/06 - POP3 (110)\|POP3]] / [[09 - Service Cheatsheets/07 - IMAP (143 993)\|IMAP]] |
| 139/445 | [[01 - enum4linux and enum4linux-ng]], [[05 - NetExec (nxc)]] |
| 161 | [[16 - snmpwalk]] |
| 389/636 | [[07 - ldapsearch]], [[09 - BloodHound]] |
| 1433 | [[33 - MSSQL Enumeration]] |
| 1521 | Oracle TNS, `odat` |
| 2049 | [[32 - NFS Enumeration]] |
| 3306 | [[34 - MySQL Enumeration]] |
| 3389 | RDP fingerprint, NLA check |
| 5985 | WinRM - test creds with [[05 - NetExec (nxc)]] |
| 6379 | [[36 - Redis Enumeration]] - often unauth |

> [!tip] Don't skip "boring" ports
> 161 (SNMP) and 2049 (NFS) regularly hand you usernames or files for free. Enumerate them every time.
