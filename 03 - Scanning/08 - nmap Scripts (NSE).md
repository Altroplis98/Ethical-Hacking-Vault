---
tags: [pentest, scanning, nmap, nse, scripts, both, recon]
tool: nmap
phase: 2
---
# nmap Scripts (NSE)

The Nmap Scripting Engine runs Lua scripts for vulnerability detection, brute-force, discovery, and more. Over 600 scripts ship with nmap.

[[03 - Scanning/00 - README|Folder index]]

## Running scripts

```bash
# Default scripts (-sC is shorthand for --script=default)
nmap -sC 10.10.10.10

# Specific script
nmap --script http-title 10.10.10.10

# Script category
nmap --script vuln 10.10.10.10

# Multiple scripts
nmap --script "http-title,http-headers" 10.10.10.10

# Wildcard
nmap --script "smb-*" 10.10.10.10

# Script with arguments
nmap --script http-brute --script-args http-brute.path=/admin/ 10.10.10.10
```

## Script categories

| Category | Purpose |
| --- | --- |
| `default` | Safe, useful scripts (what `-sC` runs) |
| `vuln` | Vulnerability checks |
| `safe` | Won't crash services |
| `intrusive` | May crash services or be noisy |
| `exploit` | Actively exploit vulnerabilities |
| `brute` | Brute-force credentials |
| `auth` | Authentication checks |
| `discovery` | Service/host discovery |
| `dos` | Denial of service (use carefully) |

## High-value scripts by service

### HTTP

```bash
nmap --script http-title -p 80 10.10.10.10
nmap --script http-enum -p 80 10.10.10.10          # Directory enumeration
nmap --script http-methods -p 80 10.10.10.10        # Allowed HTTP methods
nmap --script http-shellshock -p 80 10.10.10.10     # Shellshock check
nmap --script http-robots.txt -p 80 10.10.10.10
```

### SMB

```bash
nmap --script smb-os-discovery -p 445 10.10.10.10
nmap --script smb-enum-shares -p 445 10.10.10.10
nmap --script smb-enum-users -p 445 10.10.10.10
nmap --script smb-vuln-* -p 445 10.10.10.10         # EternalBlue, etc.
```

### General vuln scan

```bash
nmap --script vuln -p 22,80,443,445 10.10.10.10
```

## Script locations

```bash
ls /usr/share/nmap/scripts/
# Search for scripts by name
ls /usr/share/nmap/scripts/ | grep smb
```

## Updating scripts

```bash
sudo nmap --script-updatedb
```

## See also

- [[05 - nmap Basics]] — core commands
- [[07 - nmap Service and Version Detection]] — version info scripts build on
