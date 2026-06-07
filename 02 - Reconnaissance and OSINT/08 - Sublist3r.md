---
tags: [pentest, recon, osint, subdomain, sublist3r, both]
tool: sublist3r
phase: 1
---
# Sublist3r

Python-based subdomain enumeration tool using search engines. Older but still functional and pre-installed on Kali.

[[02 - Reconnaissance and OSINT/00 - README|Folder index]]

## Install / verify

```bash
# Usually pre-installed on Kali
which sublist3r || sudo apt install sublist3r -y
```

## Usage

```bash
# Basic enumeration
sublist3r -d example.com

# Save output
sublist3r -d example.com -o subs.txt

# Include brute-force
sublist3r -d example.com -b

# Custom brute-force wordlist
sublist3r -d example.com -b -t 50

# Specific ports check
sublist3r -d example.com -p 80,443,8080

# Verbose
sublist3r -d example.com -v
```

## Limitations

- Relies heavily on search engine scraping — results vary by region and rate limits
- No API key support for premium data sources
- Less maintained than subfinder/amass
- Still useful as a quick cross-check against other tools

## See also

- [[04 - Subfinder]] — faster, more sources
- [[05 - Amass]] — most comprehensive
