---
tags: [pentest, enumeration, smb, enum4linux, both, recon]
tool: enum4linux, enum4linux-ng
phase: 3
---
# enum4linux / enum4linux-ng

All-in-one SMB/NetBIOS/LDAP enumeration. Extracts users, groups, shares, password policy, OS info from Windows/Samba hosts.

[[04 - Enumeration/00 - README|Folder index]]

## enum4linux (classic)

```bash
# Full enumeration
enum4linux -a 10.10.10.10

# With credentials
enum4linux -u user -p pass -a 10.10.10.10

# Specific enum types
enum4linux -U 10.10.10.10    # Users
enum4linux -S 10.10.10.10    # Shares
enum4linux -G 10.10.10.10    # Groups
enum4linux -P 10.10.10.10    # Password policy
enum4linux -o 10.10.10.10    # OS info
```

## enum4linux-ng (modern rewrite)

```bash
# Install
pip install enum4linux-ng --break-system-packages

# Full enumeration
enum4linux-ng -A 10.10.10.10

# With credentials
enum4linux-ng -u user -p pass -A 10.10.10.10

# JSON output
enum4linux-ng -A 10.10.10.10 -oJ output.json
```

## What it extracts

| Data | Flag | Value |
| --- | --- | --- |
| Users | `-U` | AD/local user list for spraying |
| Groups | `-G` | Group memberships |
| Shares | `-S` | Accessible file shares |
| Password policy | `-P` | Lockout threshold for safe spraying |
| OS info | `-o` | OS version, SMB version |
| RID cycling | `-r` | Enumerate users via SID brute-force |

## RID cycling (when null session works)

```bash
# enum4linux RID cycling
enum4linux -r 10.10.10.10

# enum4linux-ng
enum4linux-ng -R 10.10.10.10
```

> [!tip] Always check the password policy before spraying
> `enum4linux -P` tells you the lockout threshold. If it's 5 attempts, don't spray more than 3.

## See also

- [[02 - smbclient]] — manual share browsing
- [[05 - NetExec (nxc)]] — more powerful enumeration
- [[04 - rpcclient]] — low-level RPC queries
