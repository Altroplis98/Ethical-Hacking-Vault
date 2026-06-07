---
tags: [pentest, transfer, scp, rsync, ssh, initial-access, linux]
type: cheatsheet
phase: 5
---
# scp and rsync

Secure file transfer over SSH. Use when you have SSH credentials or keys to the target.

[[00 - README|Folder index]]

## scp — Secure Copy

```bash
# Upload to target
scp linpeas.sh user@TARGET:/tmp/

# Download from target
scp user@TARGET:/etc/shadow ./shadow.txt

# Recursive (directory)
scp -r /opt/tools user@TARGET:/tmp/tools

# Non-standard SSH port
scp -P 2222 file.txt user@TARGET:/tmp/

# With SSH key
scp -i id_rsa file.txt user@TARGET:/tmp/
```

## rsync — Sync (better for large/many files)

```bash
# Upload
rsync -avz /opt/tools/ user@TARGET:/tmp/tools/

# Download
rsync -avz user@TARGET:/var/log/ ./logs/

# Non-standard port
rsync -avz -e 'ssh -p 2222' file.txt user@TARGET:/tmp/

# Dry run (preview)
rsync -avzn /opt/tools/ user@TARGET:/tmp/tools/
```

| rsync flag | Meaning |
| --- | --- |
| `-a` | Archive mode (preserves permissions, symlinks, etc.) |
| `-v` | Verbose |
| `-z` | Compress during transfer |
| `-n` | Dry run — show what would be transferred |
| `--progress` | Show transfer progress |

## When to use which

| Scenario | Tool |
| --- | --- |
| Single file | scp |
| Many files / directory | rsync |
| Resume interrupted transfer | rsync |
| Exfiltrate logs | rsync |
| Quick tool upload | scp |

## See also

- [[12 - Python HTTP Server Transfer]]
- [[19 - FTP Client Commands]]
