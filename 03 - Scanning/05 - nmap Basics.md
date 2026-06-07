---
tags: [pentest, scanning, nmap, ports, both, recon]
tool: nmap
phase: 2
---
ports=$(nmap -p- --min-rate 5000 -T4 -Pn 10.129.3.94 | grep ^[0-9] | cut -d/ -f1 | tr '\n' ',' | sed 's/,$//')

nmap -sC -sV -p $ports 10.129.3.3

nmap --script vuln -p $ports 10.129.3.3

#!/bin/bash
TARGET=10.129.3.180

ports=$(nmap -p- --min-rate 5000 -T4 -Pn $TARGET | grep ^[0-9] | cut -d/ -f1 | tr '\n' ',' | sed 's/,$//')

cat > /tmp/scan.rc <<EOF
db_nmap -O -sC -sV --script vuln -p $ports $TARGET
analyze
EOF

sudo msfdb init && msfconsole -r /tmp/scan.rc

Save it as scan.sh, make it executable with chmod +x scan.sh, then run with ./scan.sh."

# nmap Basics

The fundamental nmap commands and concepts. Start here if you're new to nmap.

[[03 - Scanning/00 - README|Folder index]]

## Core syntax

```bash
nmap [scan type] [options] <target>
```

## Target specification

```bash
nmap 10.10.10.10                  # Single IP
nmap 10.10.10.0/24                # CIDR range
nmap 10.10.10.1-100               # IP range
nmap 10.10.10.1,5,10              # Specific IPs
nmap -iL targets.txt              # From file
nmap example.com                  # Hostname
nmap --exclude 10.10.10.5         # Exclude an IP
nmap --excludefile exclude.txt    # Exclude from file
```

## The two-phase approach (recommended)

```bash
# Phase 1: Fast full-port discovery
sudo nmap -p- --min-rate 5000 -Pn -oG ports.gnmap 10.10.10.10

# Extract open ports
PORTS=$(grep -oP '\d+/open' ports.gnmap | cut -d/ -f1 | tr '\n' ',' | sed 's/,$//')

# Phase 2: Deep scan on open ports only
sudo nmap -sC -sV -p$PORTS -Pn -oA full 10.10.10.10
```

This is dramatically faster than running `-sC -sV` against all 65535 ports.

## Common scan recipes

```bash
# Quick scan — top 1000 ports
nmap 10.10.10.10

# All ports
nmap -p- 10.10.10.10

# Specific ports
nmap -p 22,80,443,445,3389 10.10.10.10

# Port range
nmap -p 1-1024 10.10.10.10

# Service version + default scripts
nmap -sC -sV 10.10.10.10

# Aggressive (OS, version, scripts, traceroute)
nmap -A 10.10.10.10

# UDP scan (top 20 UDP ports)
sudo nmap -sU --top-ports 20 10.10.10.10