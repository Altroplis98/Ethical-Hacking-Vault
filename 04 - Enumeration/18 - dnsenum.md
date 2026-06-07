---
tags: [pentest, enumeration, dns, dnsenum, recon]
tool: dnsenum
phase: 3
---
# dnsenum

DNS enumeration tool — zone transfers, brute-force, Google scraping, reverse lookups, WHOIS on netranges.

[[04 - Enumeration/00 - README|Folder index]]

## Usage

```bash
# Full enumeration
dnsenum example.com

# With subdomain brute-force
dnsenum --enum -f /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt example.com

# Specify DNS server
dnsenum --dnsserver 8.8.8.8 example.com

# More threads for brute-force
dnsenum --threads 50 example.com

# Output
dnsenum -o output.xml example.com
```

> Detailed coverage in [[02 - Reconnaissance and OSINT/09 - dnsenum dnsrecon fierce|dnsenum dnsrecon fierce]] — this note exists for cross-reference from the enumeration phase.

## See also

- [[19 - dnsrecon]] — more structured DNS enum
- [[20 - fierce]] — DNS recon and adjacent IP discovery
