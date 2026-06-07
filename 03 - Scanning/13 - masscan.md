---
tags: [pentest, scanning, masscan, ports, both, recon]
tool: masscan
phase: 2
---
# masscan

The fastest port scanner — scans the entire internet in under 6 minutes. Uses its own TCP/IP stack for raw speed.

[[03 - Scanning/00 - README|Folder index]]

## Install / verify

```bash
which masscan || sudo apt install masscan -y
```

## Usage

```bash
# Scan top ports on a /24
sudo masscan 10.10.10.0/24 -p 80,443,22,445,3389 --rate 1000

# Full port scan on a single host
sudo masscan 10.10.10.10 -p 0-65535 --rate 1000

# Common web ports on a large range
sudo masscan 10.0.0.0/8 -p 80,443,8080,8443 --rate 10000

# Output formats
sudo masscan 10.10.10.0/24 -p 1-65535 --rate 1000 -oG masscan.gnmap
sudo masscan 10.10.10.0/24 -p 1-65535 --rate 1000 -oX masscan.xml
sudo masscan 10.10.10.0/24 -p 1-65535 --rate 1000 -oL masscan.list

# Exclude targets
sudo masscan 10.10.10.0/24 -p 80 --excludefile exclude.txt
```

## Key flags

| Flag | Purpose |
| --- | --- |
| `-p ports` | Port specification |
| `--rate N` | Packets per second |
| `--banners` | Grab service banners (slower) |
| `-oG file` | Grepable output |
| `-oX file` | XML output |
| `-oL file` | List output |
| `--excludefile` | Exclude IPs from file |
| `--adapter-ip` | Source IP |
| `--adapter-port` | Source port |
| `--wait N` | Seconds to wait after scan for late responses |

## masscan → nmap pipeline

```bash
# Fast discovery with masscan, deep scan with nmap
sudo masscan 10.10.10.0/24 -p 1-65535 --rate 1000 -oL masscan.list

# Parse masscan output → feed to nmap
grep '^open' masscan.list | awk '{print $4":"$3}' | sort -u > targets_ports.txt
# Then scan each host:port combination with nmap -sC -sV
```

> [!warning] Rate limiting
> `--rate 10000` on a client network can cause network disruption. Start with `--rate 1000` and increase if the network handles it.

## See also

- [[14 - RustScan]] — similar speed, better nmap integration
- [[05 - nmap Basics]] — for deep scanning after masscan discovery
