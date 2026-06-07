---
tags: [pentest, enumeration, kerberos, kerbrute, ad, active-directory, recon, windows]
tool: kerbrute
phase: 3
---
# kerbrute

Fast Kerberos user enumeration and password brute-forcing. Uses Kerberos pre-authentication to validate usernames without triggering traditional logon events.

[[04 - Enumeration/00 - README|Folder index]]

## Install

```bash
# Download from GitHub releases
wget https://github.com/ropnop/kerbrute/releases/latest/download/kerbrute_linux_amd64
chmod +x kerbrute_linux_amd64
sudo mv kerbrute_linux_amd64 /usr/local/bin/kerbrute
```

## User enumeration

```bash
# Enumerate valid usernames (doesn't lock accounts)
kerbrute userenum -d corp.local --dc 10.10.10.10 usernames.txt

# With threading
kerbrute userenum -d corp.local --dc 10.10.10.10 -t 50 usernames.txt
```

## Password spraying

```bash
# Single password against user list
kerbrute passwordspray -d corp.local --dc 10.10.10.10 users.txt 'Spring2024!'

# Brute-force (all combos)
kerbrute bruteuser -d corp.local --dc 10.10.10.10 passwords.txt username
```

## Why kerbrute for enumeration

- Valid usernames return `KDC_ERR_PREAUTH_REQUIRED`
- Invalid usernames return `KDC_ERR_C_PRINCIPAL_UNKNOWN`
- **Does NOT generate Windows logon events** for enumeration (only pre-auth errors)
- Faster than LDAP enumeration

> [!warning] Password spraying DOES generate logon events
> Only enumeration is stealthy. Spraying creates event 4771 (Kerberos pre-auth failure).

## See also

- [[13 - Impacket GetNPUsers (AS-REP)]] — find AS-REP roastable users
- [[14 - Impacket GetUserSPNs (Kerberoast)]] — find kerberoastable users
