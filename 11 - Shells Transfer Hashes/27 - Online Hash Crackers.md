---
tags: [pentest, cracking, online, rainbow-tables, both, initial-access]
type: reference
phase: 7
---
# Online Hash Crackers

When you don't have GPU hardware or need a quick lookup for common hashes.

[[00 - README|Folder index]]

## Free lookup services

| Service | URL | Notes |
| --- | --- | --- |
| CrackStation | crackstation.net | Lookup table for unsalted hashes |
| Hashes.com | hashes.com | Community database |
| cmd5.org | cmd5.org | MD5/SHA lookups |
| OnlineHashCrack | onlinehashcrack.com | Paid service for WPA, NTLM, etc. |

## When online crackers work

- **Unsalted hashes**: MD5, SHA1, SHA256, NTLM — if the password is common, it's in the rainbow table
- **Quick CTF lookups**: Paste a hash, get instant result

## When they don't work

- **Salted hashes**: bcrypt, sha512crypt, scrypt — each salt makes the hash unique
- **Uncommon passwords**: Only pre-computed or previously cracked passwords are in the database
- **WPA**: Some services accept .cap files but charge for cracking

> [!warning] OPSEC
> Uploading client hashes to public crackers is a confidentiality risk. Only use for CTFs, labs, or your own hashes. In real engagements, crack locally.

## See also

- [[20 - Hashcat Mode IDs]]
- [[24 - John the Ripper Reference]]
- [[28 - GPU Setup for Hashcat]]
