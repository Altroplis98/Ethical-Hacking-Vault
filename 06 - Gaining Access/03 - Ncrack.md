---
tags: [pentest, brute-force, ncrack, credentials, initial-access]
tool: ncrack
phase: 5
---
# Ncrack

From the Nmap project. Often the best choice for **RDP brute / spray** because it handles NLA cleanly.

[[06 - Gaining Access/00 - README|Folder index]]

## Syntax

```bash
ncrack -p <port> --user <u> -P pass.txt <ip>
ncrack -p <port> -U users.txt -P pass.txt <ip>
```

| Flag | Meaning |
| --- | --- |
| `-p PORT` | service port |
| `--user U` | single user |
| `-U list` | users list |
| `-p list` (capital P) | password list |
| `-T0..5` | timing (5=insane) |
| `-f` | stop on first valid |
| `-oN out.txt` | normal output |
| `-oX out.xml` | XML output |

## Recipes

### RDP

```bash
ncrack -vv --user administrator -P pass.txt rdp://<ip>
ncrack -vv -U users.txt -P pass.txt rdp://<ip>:3389
```

### SSH

```bash
ncrack -p 22 --user user -P pass.txt <ip> -T 4
```

### SMB

```bash
ncrack -p 445 --user administrator -P pass.txt <ip>
```

### Many services in one go

```bash
ncrack -vv -U users.txt -P pass.txt ssh://<ip> ftp://<ip> smb://<ip>
```

## When to pick Ncrack over Hydra

- RDP with NLA (Hydra often hangs)
- You want XML output for tool chains
- You're already using nmap and want consistent UX

## When NOT to use

- Windows AD password spraying - use [[04 - NetExec Password Attacks]] which respects lockout policy
