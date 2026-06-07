---
tags: [pentest, enumeration, smb, smbclient, recon, windows]
tool: smbclient
phase: 3
---
# smbclient

FTP-like client for SMB shares. Browse, download, and upload files to Windows/Samba shares.

[[04 - Enumeration/00 - README|Folder index]]

## List shares

```bash
# Null session (anonymous)
smbclient -L //10.10.10.10 -N

# With credentials
smbclient -L //10.10.10.10 -U 'user%password'

# Domain credentials
smbclient -L //10.10.10.10 -U 'DOMAIN/user%password'
```

## Connect to a share

```bash
# Anonymous
smbclient //10.10.10.10/share -N

# With creds
smbclient //10.10.10.10/share -U 'user%password'
```

## Interactive commands

```text
smb: \> ls                    # List files
smb: \> cd directory          # Change directory
smb: \> get file.txt          # Download file
smb: \> put local.txt         # Upload file
smb: \> mget *.txt            # Download multiple
smb: \> recurse ON            # Enable recursive operations
smb: \> prompt OFF            # Disable per-file prompts
smb: \> mget *                # Download everything recursively
```

## One-liner download

```bash
# Download a specific file
smbclient //10.10.10.10/share -N -c 'get path/to/file.txt'

# Download everything
smbclient //10.10.10.10/share -N -c 'recurse ON; prompt OFF; mget *'
```

## See also

- [[03 - smbmap]] — permission mapping (better overview than smbclient)
- [[01 - enum4linux and enum4linux-ng]] — broader SMB enumeration
