---
tags: [pentest, recon, dns, dnsenum, dnsrecon, fierce, both]
phase: 1
---
# dnsenum / dnsrecon / fierce

Three classic DNS enumeration tools that automate zone transfers, brute-forcing, and record collection.

[[02 - Reconnaissance and OSINT/00 - README|Folder index]]

## dnsenum

All-in-one DNS enumeration: host resolution, zone transfers, Google scraping, brute-force.

```bash
# Basic enum
dnsenum example.com

# With brute-force
dnsenum --enum -f /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt example.com

# Key flags
dnsenum --dnsserver 8.8.8.8 example.com  # Use specific resolver
dnsenum --threads 50 example.com          # Faster brute-force
dnsenum -o output.xml example.com         # XML output
```

## dnsrecon

More structured output, supports multiple enum modes.

```bash
# Standard enumeration (NS, MX, SOA, zone transfer attempt)
dnsrecon -d example.com

# Brute-force subdomains
dnsrecon -d example.com -t brt -D /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt

# Reverse lookup sweep
dnsrecon -r 203.0.113.0/24 -t rvl

# Zone transfer
dnsrecon -d example.com -t axfr

# SRV record enumeration
dnsrecon -d example.com -t srv

# Cache snooping
dnsrecon -t snoop -n <dns_server> -D /path/to/domains.txt

# Output formats
dnsrecon -d example.com -j output.json  # JSON
dnsrecon -d example.com --xml output.xml  # XML
```

### dnsrecon enum types

| `-t` mode | What it does |
| --- | --- |
| `std` | Standard — NS, MX, SOA, AXFR attempt |
| `brt` | Brute-force subdomains |
| `rvl` | Reverse lookups on a range |
| `axfr` | Zone transfer |
| `srv` | SRV record enum |
| `snoop` | DNS cache snooping |
| `zonewalk` | DNSSEC zone walking |

## fierce

Lightweight DNS reconnaissance and brute-forcer. Good for quick sweeps.

```bash
# Basic scan
fierce --domain example.com

# Custom wordlist
fierce --domain example.com --subdomain-file /path/to/wordlist.txt

# Specify DNS server
fierce --domain example.com --dns-servers 8.8.8.8

# Wide scan (expand to nearby IPs)
fierce --domain example.com --wide
```

## When to use which

| Tool | Best for |
| --- | --- |
| dnsenum | Quick all-in-one with Google scraping |
| dnsrecon | Structured output, multiple enum modes, reporting |
| fierce | Fast initial DNS sweep, discovering adjacent IP space |

## See also

- [[02 - DNS Recon Basics]] — manual dig commands
- [[04 - Subfinder]] — passive subdomain discovery
