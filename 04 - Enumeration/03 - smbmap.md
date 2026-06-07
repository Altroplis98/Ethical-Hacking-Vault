---
tags: [pentest, enumeration, smb, smbmap, recon, windows]
tool: smbmap
phase: 3
---
# smbmap

SMB share enumeration with permission mapping. Shows read/write access at a glance.

[[04 - Enumeration/00 - README|Folder index]]

## Usage

```bash
# Null session
smbmap -H 10.10.10.10

# With credentials
smbmap -H 10.10.10.10 -u user -p password

# Domain creds
smbmap -H 10.10.10.10 -u user -p password -d DOMAIN

# List contents of a share
smbmap -H 10.10.10.10 -r 'share_name'

# Recursive listing
smbmap -H 10.10.10.10 -R 'share_name'

# Download a file
smbmap -H 10.10.10.10 -u user -p pass --download 'share\path\file.txt'

# Upload a file
smbmap -H 10.10.10.10 -u user -p pass --upload local.txt 'share\path\remote.txt'

# Execute a command (if writable + admin)
smbmap -H 10.10.10.10 -u admin -p pass -x 'ipconfig'
```

## Output reading

```text
[+] IP: 10.10.10.10:445   Name: target
  Disk          Permissions     Comment
  ----          -----------     -------
  ADMIN$        NO ACCESS
  C$            NO ACCESS
  IPC$          READ ONLY
  Users         READ ONLY
  Backups       READ, WRITE     Backup share
```

## Key flags

| Flag | Purpose |
| --- | --- |
| `-H host` | Target host |
| `-u user` | Username |
| `-p pass` | Password |
| `-d domain` | Domain |
| `-r share` | List share contents |
| `-R share` | Recursive listing |
| `--download` | Download a file |
| `--upload` | Upload a file |
| `-x cmd` | Execute command |
| `-s share` | Specify share |

## See also

- [[02 - smbclient]] — interactive share browsing
- [[05 - NetExec (nxc)]] — broadest SMB capabilities
