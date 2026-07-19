---
tags: [pentest, cracking, hashcat, modes, reference, both, initial-access]
tool: hashcat
phase: 7
---
# Hashcat Mode IDs

Quick-reference table for the most common hashcat `-m` mode numbers.

[[11 - Shells Transfer Hashes/00 - README|Folder index]]

## Linux / Unix

| Mode | Hash type |
| --- | --- |
| 500 | md5crypt `$1$` |
| 1800 | sha512crypt `$6$` (Linux shadow) |
| 7400 | sha256crypt `$5$` |
| 3200 | bcrypt `$2*$` |
| 1500 | DES (descrypt) — older Unix `/etc/passwd` |
| 6300 | AIX `{smd5}` |
| 1100 | Domain Cached Credentials v1 (DCC1) |

## Windows

| Mode | Hash type |
| --- | --- |
| 1000 | NTLM |
| 3000 | LM |
| 5500 | NetNTLMv1 / MSCHAPv2 |
| 5600 | NetNTLMv2 |
| 13100 | Kerberoast (TGS-REP etype 23) |
| 18200 | AS-REP Roast (etype 23) |
| 19600 | Kerberos 5 TGS-REP etype 17 (AES128) |
| 19700 | Kerberos 5 TGS-REP etype 18 (AES256) |
| 2100 | DCC2 / mscash2 (domain cached credentials — offline domain machines) |
| 26500 | Kerberos 5 AS-REQ Pre-Auth etype 23 (kerbrute output) |
| 22100 | BitLocker |

## Web / Application

| Mode | Hash type |
| --- | --- |
| 0 | MD5 |
| 100 | SHA1 |
| 1400 | SHA-256 |
| 1700 | SHA-512 |
| 10 | md5($pass.$salt) |
| 20 | md5($salt.$pass) |
| 3200 | bcrypt |
| 400 | phpass (WordPress, phpBB) |
| 7900 | Drupal7 |
| 3710 | md5($salt.md5($pass)) — common custom web apps |
| 10000 | Django PBKDF2-SHA256 |
| 1711 | SSHA-512 (LDAP) |
| 10900 | PBKDF2-HMAC-SHA256 (generic) |

## Network / Wireless

| Mode | Hash type |
| --- | --- |
| 22000 | WPA-PBKDF2-PMKID+EAPOL |
| 5500 | NetNTLMv1 / MSCHAPv2 |
| 5600 | NetNTLMv2 |
| 16800 | WPA-PMKID (deprecated — use 22000) |

## Database

| Mode | Hash type |
| --- | --- |
| 300 | MySQL4.1/5 |
| 1731 | MSSQL (2012+) |
| 12 | PostgreSQL |
| 200 | MySQL323 |

## Archives / Documents

| Mode | Hash type |
| --- | --- |
| 13600 | WinZip |
| 17200 | PKZIP |
| 13000 | RAR5 |
| 12500 | RAR3 |
| 11600 | 7-Zip |
| 9400 | MS Office 2007 (`$office$*2007*`) |
| 9500 | MS Office 2010 (`$office$*2010*`) |
| 9600 | MS Office 2013/2016/2019/365 (`$office$*2013*`) — very slow (100k PBKDF2 iterations) |
| 10400 | PDF 1.1–1.3 |
| 10500 | PDF 1.4–1.6 |
| 13400 | KeePass 1/2 |
| 16900 | Ansible Vault |
| 15500 | JKS (Java Keystore) |
| 15300 | DPAPI masterkey v1 |

## Mobile / Misc

| Mode | Hash type |
| --- | --- |
| 8800 | Android FDE (Samsung) |
| 11300 | Bitcoin/Litecoin wallet |

> [!tip] Find the right mode
> ```bash
> hashcat --help | grep -i "ntlm"
> hashcat -m 0 --example-hashes   # show example hash format
> ```

## See also

- [[21 - Hashcat Attack Modes]]
- [[22 - Hashcat Masks]]
- [[23 - Hashcat Rules]]
- [[26 - Hash Identification]]
