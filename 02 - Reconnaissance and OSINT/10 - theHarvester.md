---
tags: [pentest, recon, osint, theharvester, email, both]
tool: theHarvester
phase: 1
---
# theHarvester

Collects emails, subdomains, hosts, employee names, open ports, and banners from public sources.

[[02 - Reconnaissance and OSINT/00 - README|Folder index]]

## Install / verify

```bash
which theHarvester || sudo apt install theharvester -y
```

## Basic usage

```bash
# Search all sources for a domain
theHarvester -d example.com -b all

# Specific sources
theHarvester -d example.com -b google,bing,linkedin,dnsdumpster

# Limit results
theHarvester -d example.com -l 200 -b all

# Output to file
theHarvester -d example.com -b all -f output  # creates output.html + output.json
```

## Key flags

| Flag | Purpose |
| --- | --- |
| `-d domain` | Target domain |
| `-b source` | Data source(s) — `all`, `google`, `bing`, `linkedin`, etc. |
| `-l N` | Limit results per source |
| `-f file` | Output file (HTML + JSON) |
| `-n` | Enable DNS resolution on discovered hosts |
| `-t` | TLD discovery (brute-force TLDs) |
| `-S start` | Start result number (pagination) |
| `-s` | Use Shodan to query discovered hosts |
| `-v` | Verify hostnames via DNS |

## Useful sources

| Source | What it finds |
| --- | --- |
| `google` | Emails, subdomains from Google search |
| `bing` | Same via Bing |
| `linkedin` | Employee names (needs API key) |
| `dnsdumpster` | Subdomains, DNS records |
| `crtsh` | Certificate Transparency subdomains |
| `threatcrowd` | Domains, IPs, emails |
| `virustotal` | Subdomains (needs API key) |
| `shodan` | Hosts, banners (needs API key) |
| `hunter` | Email addresses (needs API key) |

## API keys

Store in `/etc/theHarvester/api-keys.yaml` or `~/.theHarvester/api-keys.yaml`:

```yaml
apikeys:
  shodan:
    key: YOUR_KEY
  hunter:
    key: YOUR_KEY
  virustotal:
    key: YOUR_KEY
```

## See also

- [[11 - Recon-ng]] — more modular OSINT framework
- [[12 - Spiderfoot]] — automated OSINT with GUI
- [[14 - Google Dorking]] — manual search-engine recon
