---
tags: [pentest, brute-force, medusa, credentials, initial-access]
tool: medusa
phase: 5
---
# Medusa

Parallel network login brute-forcer. Alternative to Hydra. Smaller protocol list but sometimes more reliable on flaky targets.

[[06 - Gaining Access/00 - README|Folder index]]

## Syntax

```bash
medusa -h <target> -U users.txt -P pass.txt -M <module>
```

| Flag | Meaning |
| --- | --- |
| `-h host` | single target |
| `-H file` | hosts list |
| `-u user` / `-U list` | single user / list |
| `-p pass` / `-P list` | single pass / list |
| `-M module` | service module |
| `-d` | list supported modules |
| `-t N` | concurrent logins per host (default 1) |
| `-T N` | concurrent hosts (default 1) |
| `-O out.txt` | log valid pairs |
| `-e ns` | try null / same-as-user |
| `-f` | stop after first valid on host |
| `-v 6` | verbose level |

## Modules

```bash
medusa -d        # list available
# Common: ssh, ftp, http, smbnt, mysql, mssql, vnc, rsh, rlogin, pop3, telnet
```

## Recipes

### SSH

```bash
medusa -h <ip> -U users.txt -P pass.txt -M ssh -t 4 -f
```

### FTP

```bash
medusa -h <ip> -U users.txt -P pass.txt -M ftp -e ns
```

### SMB

```bash
medusa -h <ip> -U users.txt -P pass.txt -M smbnt -t 4
```

### MSSQL

```bash
medusa -h <ip> -u sa -P pass.txt -M mssql -t 8
```

### Web (HTTP basic)

```bash
medusa -h <ip> -U users.txt -P pass.txt -M http -m DIR:/admin/ -t 4
```

## Hydra vs Medusa - when to prefer which

| Need | Pick |
| --- | --- |
| HTTP-POST form with `<title>` failure | Hydra (better form module) |
| RDP | Hydra or Ncrack |
| SSH at scale | Hydra (better threading) |
| Slow/unstable target | Medusa (more graceful) |
| MS-SQL / MySQL big password list | Medusa |
| Domain accounts | Neither - use [[04 - NetExec Password Attacks]] |

> [!tip] One gotcha
> Medusa's `-t` (per-host) and `-T` (across-hosts) are separate; Hydra's `-t` is global. Don't mix them up - you may DoS yourself with too many threads.
