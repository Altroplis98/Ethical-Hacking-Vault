---
tags: [pentest, scanning, fping, host-discovery, icmp, both, recon]
tool: fping
phase: 2
---
# fping

ICMP-based host discovery that pings multiple hosts in parallel. Much faster than `ping` for sweeping ranges.

[[03 - Scanning/00 - README|Folder index]]

## Install / verify

```bash
which fping || sudo apt install fping -y
```

## Usage

```bash
# Sweep a /24 — show only alive hosts
fping -a -g 10.10.10.0/24 2>/dev/null

# Generate alive host list
fping -a -g 10.10.10.0/24 2>/dev/null > alive.txt

# Show unreachable too
fping -a -u -g 10.10.10.0/24

# From a file of IPs
fping -a -f targets.txt 2>/dev/null

# Custom retry and timeout
fping -a -g 10.10.10.0/24 -r 1 -t 100 2>/dev/null
```

## Key flags

| Flag | Purpose |
| --- | --- |
| `-a` | Show only alive hosts |
| `-u` | Show only unreachable hosts |
| `-g` | Generate target list from CIDR |
| `-f file` | Read targets from file |
| `-r N` | Retry count (default 3) |
| `-t N` | Timeout per probe in ms |
| `-c N` | Ping count per host |
| `-q` | Quiet — only summary |

## Limitations

- ICMP-only — hosts with ICMP blocked won't respond
- Many cloud/hardened hosts drop ICMP
- Use ARP scan or TCP ping as fallback

## See also

- [[01 - arp-scan]] — ARP-based (local subnet, bypasses ICMP blocks)
- [[04 - nmap Host Discovery]] — multi-method discovery
