---
tags: [pentest, recon, osint, spiderfoot, both]
tool: spiderfoot
phase: 1
---
# Spiderfoot

Automated OSINT collection tool with a web UI. Correlates data across 200+ sources.

[[02 - Reconnaissance and OSINT/00 - README|Folder index]]

## Install / verify

```bash
# Kali
which spiderfoot || sudo apt install spiderfoot -y

# Start web UI
spiderfoot -l 127.0.0.1:5001
# Browse to http://127.0.0.1:5001
```

## CLI mode

```bash
# Run a scan from CLI
spiderfoot -s example.com -t all -o csv > results.csv

# Specific scan types
spiderfoot -s example.com -m sfp_dnsresolve,sfp_shodan -o csv
```

## Key scan types

| Type | What it checks |
| --- | --- |
| All | Everything (slow but thorough) |
| Footprint | Infrastructure mapping |
| Investigate | Deep dive on a target |
| Passive | No direct target contact |

## Useful modules

| Module | Purpose |
| --- | --- |
| `sfp_dnsresolve` | DNS resolution and brute-force |
| `sfp_shodan` | Shodan data integration |
| `sfp_haveibeenpwned` | Breach data lookup |
| `sfp_whois` | WHOIS data |
| `sfp_certspotter` | Certificate Transparency |
| `sfp_github` | GitHub code search |
| `sfp_socialprofiles` | Social media profiles |

## Web UI workflow

1. New Scan → enter target (domain, IP, email, name)
2. Select scan type (Footprint for broad, Passive for stealth)
3. Configure API keys in Settings → API Keys
4. Start scan → monitor progress in real-time
5. Browse results by data type (emails, hosts, credentials, etc.)
6. Export results as CSV, JSON, or GEXF (graph format)

## See also

- [[11 - Recon-ng]] — CLI-based modular alternative
- [[13 - Maltego]] — commercial graph-based OSINT
