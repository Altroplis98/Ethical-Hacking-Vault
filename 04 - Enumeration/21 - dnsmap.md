---
tags: [pentest, enumeration, dns, dnsmap, recon]
tool: dnsmap
phase: 3
---
# dnsmap

Simple DNS brute-forcer. Tries subdomains from a built-in or custom wordlist.

[[04 - Enumeration/00 - README|Folder index]]

## Install / verify

```bash
which dnsmap || sudo apt install dnsmap -y
```

## Usage

```bash
# Default wordlist
dnsmap example.com

# Custom wordlist
dnsmap example.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt

# Output
dnsmap example.com -r results.txt

# Delay between queries (ms)
dnsmap example.com -d 300
```

## When to use

dnsmap is simpler than dnsrecon/dnsenum — use it for a quick brute-force when you don't need zone transfers or reverse lookups. For comprehensive DNS enum, prefer [[19 - dnsrecon]].

## See also

- [[19 - dnsrecon]] — full-featured DNS enumeration
- [[18 - dnsenum]] — another comprehensive option
