---
tags: [pentest, scanning, nmap, version-detection, both, recon]
tool: nmap
phase: 2
---
# nmap Service and Version Detection

Using `-sV` to identify exact services and versions running on open ports. Critical for vulnerability mapping.

[[03 - Scanning/00 - README|Folder index]]

## Basic version detection

```bash
nmap -sV 10.10.10.10
nmap -sV -p 22,80,443 10.10.10.10
```

## Version intensity

```bash
# Light probe (fast, less accurate)
nmap -sV --version-intensity 2 10.10.10.10

# Default intensity (level 7)
nmap -sV 10.10.10.10

# Maximum intensity (slow, most accurate)
nmap -sV --version-intensity 9 10.10.10.10

# Version all (alias for intensity 9)
nmap -sV --version-all 10.10.10.10
```

## OS detection

```bash
# OS detection
sudo nmap -O 10.10.10.10

# OS detection with version
sudo nmap -O -sV 10.10.10.10

# Aggressive OS detection (more guesses)
sudo nmap -O --osscan-guess 10.10.10.10

# Combined aggressive scan
sudo nmap -A 10.10.10.10   # -sV + -O + -sC + traceroute
```

## What version detection reveals

```text
PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http        Apache httpd 2.4.41 ((Ubuntu))
443/tcp  open  ssl/http    nginx 1.18.0
445/tcp  open  netbios-ssn Samba smbd 4.6.2
3306/tcp open  mysql       MySQL 5.7.38-0ubuntu0.18.04.1
```

This output directly maps to:
- `searchsploit OpenSSH 8.2` → check for exploits
- `searchsploit Apache 2.4.41` → check for exploits
- OS identification: Ubuntu 18.04 based on package versions

## Tips

- Always run `-sV` on discovered ports — raw port numbers tell you nothing about non-standard services
- `-sV` can sometimes trigger IDS alerts (sends protocol-specific probes)
- Pair with `-sC` for default scripts that often extract additional version info

## See also

- [[05 - nmap Basics]] — core commands
- [[08 - nmap Scripts (NSE)]] — scripts that complement version detection
