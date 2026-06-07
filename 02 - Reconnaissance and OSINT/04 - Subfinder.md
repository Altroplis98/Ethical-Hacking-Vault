---
tags: [pentest, recon, osint, subdomain, subfinder, both]
tool: subfinder
phase: 1
---
# Subfinder

Fast passive subdomain enumeration tool by ProjectDiscovery. Queries CT logs, search engines, DNS datasets, and more without touching the target.

[[02 - Reconnaissance and OSINT/00 - README|Folder index]]

## Install / verify

```bash
# Kali (may need manual install)
go install -v github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest

# Or via apt (if available)
sudo apt install subfinder -y

subfinder -version
```

## Basic usage

```bash
# Single domain
subfinder -d example.com

# Silent mode (clean output, one subdomain per line)
subfinder -d example.com -silent

# Multiple domains
subfinder -dL domains.txt -silent

# Output to file
subfinder -d example.com -o subs.txt
```

## Key flags

| Flag | Purpose |
| --- | --- |
| `-d domain` | Target domain |
| `-dL file` | List of target domains |
| `-silent` | Only print subdomains (no banner/stats) |
| `-o file` | Output file |
| `-oJ` | JSON output |
| `-nW` | Remove wildcard subdomains |
| `-t N` | Concurrency threads (default 10) |
| `-timeout N` | Seconds per source timeout |
| `-all` | Use all sources (slower but more complete) |
| `-sources` | List available sources |
| `-recursive` | Recursive subdomain enum |
| `-cs` | Show source for each subdomain |

## API keys (dramatically increases results)

Configure in `~/.config/subfinder/provider-config.yaml`:

```yaml
censys:
  - ac_key:ac_secret
chaos:
  - your_chaos_key
github:
  - ghp_your_token
shodan:
  - your_shodan_key
securitytrails:
  - your_st_key
virustotal:
  - your_vt_key
```

> [!tip] Even free-tier API keys help
> Shodan, VirusTotal, and SecurityTrails all have free tiers. Adding them can double your subdomain count.

## Workflow integration

```bash
# Subfinder → httpx (find live web servers)
subfinder -d example.com -silent | httpx -silent -title -status-code

# Subfinder → naabu (port scan live hosts)
subfinder -d example.com -silent | naabu -silent -top-ports 100

# Subfinder → nuclei (vuln scan discovered hosts)
subfinder -d example.com -silent | httpx -silent | nuclei -t cves/
```

## See also

- [[05 - Amass]] — more comprehensive but slower
- [[06 - Assetfinder]] — simpler alternative
- [[03 - Certificate Transparency]] — one of subfinder's data sources
