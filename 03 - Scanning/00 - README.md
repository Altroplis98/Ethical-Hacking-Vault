---
tags: [pentest, scanning, moc, both, recon]
phase: 2
---
# 03 - Scanning

Live hosts, open ports, services, OS, banners. Go loud only as your ROE allows.

[[00 - Vault Index|Home]] · Prev: [[02 - Reconnaissance and OSINT/00 - README|Recon]] · Next: [[04 - Enumeration/00 - README|Enumeration]]

## Files in this folder

### Host discovery
- [[01 - arp-scan]]
- [[02 - netdiscover]]
- [[03 - fping]]
- [[04 - nmap Host Discovery]]

### nmap deep dive
- [[05 - nmap Basics]]
- [[06 - nmap Port Scanning]]
- [[07 - nmap Service and Version Detection]]
- [[08 - nmap Scripts (NSE)]]
- [[09 - nmap Output Formats]]
- [[10 - nmap Firewall Evasion and Decoys]]
- [[11 - nmap Timing Templates]]
- [[12 - nmap Idle (Zombie) Scan]]

### Mass scanners
- [[13 - masscan]]
- [[14 - RustScan]]
- [[15 - Naabu]]

### Web fingerprinting
- [[16 - WhatWeb]]
- [[17 - wafw00f]]
- [[18 - httpx]]
- [[19 - Nuclei Tech Detection]]

### Cloud surface
- [[20 - ScoutSuite]]
- [[21 - Prowler AWS]]
- [[22 - ROADrecon Azure]]

## Two-phase scan strategy

```text
Phase 1: Discovery (fast)
    sudo nmap -p- --min-rate 5000 -Pn target -oG ports.gnmap
    (or masscan / rustscan for /16+ ranges)

Phase 2: Targeted scripts on the OPEN ports only
    OPEN=$(grep -oP '\d+/open' ports.gnmap | cut -d/ -f1 | tr '\n' ',')
    sudo nmap -sC -sV -p$OPEN -Pn -oA full target
```

> [!tip] Always keep raw output
> Use `-oA basename` (saves .nmap, .gnmap, .xml). You'll need .gnmap for piping to other tools and .xml for report generators.
