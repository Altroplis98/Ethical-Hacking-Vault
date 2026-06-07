---
tags: [pentest, enumeration, dns, dnsrecon, recon]
tool: dnsrecon
phase: 3
---
# dnsrecon

Structured DNS enumeration — zone transfers, brute-force, SRV records, cache snooping, DNSSEC zone walking.

[[04 - Enumeration/00 - README|Folder index]]

## Usage

```bash
# Standard enumeration
dnsrecon -d example.com

# Brute-force
dnsrecon -d example.com -t brt -D /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt

# Zone transfer
dnsrecon -d example.com -t axfr

# Reverse lookup sweep
dnsrecon -r 10.10.10.0/24 -t rvl

# SRV records
dnsrecon -d example.com -t srv

# DNSSEC zone walk
dnsrecon -d example.com -t zonewalk

# Output
dnsrecon -d example.com -j output.json
```

> Full details in [[02 - Reconnaissance and OSINT/09 - dnsenum dnsrecon fierce|dnsenum dnsrecon fierce]].

## See also

- [[18 - dnsenum]] — alternative DNS enum
- [[21 - dnsmap]] — pure DNS brute-forcer
