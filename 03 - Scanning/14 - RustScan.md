---
tags: [pentest, scanning, rustscan, ports, both, recon]
tool: rustscan
phase: 2
---
# RustScan

Fast port scanner written in Rust that automatically pipes results to nmap. Best of both worlds: masscan speed + nmap depth.

[[03 - Scanning/00 - README|Folder index]]

## Install

```bash
# Docker (easiest)
docker pull rustscan/rustscan

# Or download from GitHub releases
wget https://github.com/RustScan/RustScan/releases/latest/download/rustscan_amd64.deb
sudo dpkg -i rustscan_amd64.deb
```

## Usage

```bash
# Basic scan → auto-pipes to nmap
rustscan -a 10.10.10.10

# Custom port range
rustscan -a 10.10.10.10 -r 1-65535

# Adjust batch size (concurrent connections)
rustscan -a 10.10.10.10 -b 1000

# Custom nmap flags after --
rustscan -a 10.10.10.10 -- -sC -sV -oA scan

# Multiple targets
rustscan -a 10.10.10.10,10.10.10.11

# From file
rustscan -a targets.txt
```

## Key flags

| Flag | Purpose |
| --- | --- |
| `-a target` | Target IP(s) |
| `-r range` | Port range |
| `-b N` | Batch size (connections at once) |
| `--ulimit N` | File descriptor limit |
| `-t N` | Timeout per port (ms) |
| `-- <nmap flags>` | Pass flags to nmap |

## Why RustScan over masscan

| Feature | RustScan | masscan |
| --- | --- | --- |
| Auto-nmap integration | Yes | No |
| Ease of use | Simple | Config needed for complex scans |
| Speed | Very fast | Fastest |
| Banner grabbing | Via nmap | Built-in (basic) |

## See also

- [[13 - masscan]] — even faster for huge ranges
- [[05 - nmap Basics]] — what RustScan feeds into
