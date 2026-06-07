---
tags: [pentest, recon, osint, recon-ng, framework, both]
tool: recon-ng
phase: 1
---
# Recon-ng

Modular web reconnaissance framework with a Metasploit-like interface. Stores results in a local database for correlation.

[[02 - Reconnaissance and OSINT/00 - README|Folder index]]

## Install / verify

```bash
which recon-ng || sudo apt install recon-ng -y
```

## First-time setup

```bash
recon-ng

# Create a workspace
[recon-ng] > workspaces create client_name

# Install modules (marketplace)
[recon-ng] > marketplace search
[recon-ng] > marketplace install all   # install everything
# Or install specific modules:
[recon-ng] > marketplace install recon/domains-hosts/hackertarget
```

## Core workflow

```bash
# 1. Seed the database with target domain
[recon-ng] > db insert domains
domain (TEXT): example.com

# 2. Run modules to discover hosts
[recon-ng] > modules load recon/domains-hosts/hackertarget
[recon-ng] > run

# 3. Check what was found
[recon-ng] > show hosts
[recon-ng] > show contacts

# 4. Run more modules against discovered data
[recon-ng] > modules load recon/hosts-hosts/resolve
[recon-ng] > run
```

## Key modules

| Module | Purpose |
| --- | --- |
| `recon/domains-hosts/hackertarget` | Subdomain discovery |
| `recon/domains-hosts/google_site_web` | Google site: scraping |
| `recon/domains-hosts/brute_hosts` | DNS brute-force |
| `recon/domains-contacts/whois_pocs` | WHOIS contact extraction |
| `recon/contacts-credentials/hibp_breach` | Check emails against HIBP |
| `recon/hosts-ports/shodan_ip` | Port info from Shodan |
| `reporting/html` | Generate HTML report |
| `reporting/csv` | CSV export |

## API keys

```bash
[recon-ng] > keys list
[recon-ng] > keys add shodan_api YOUR_KEY
[recon-ng] > keys add virustotal_api YOUR_KEY
[recon-ng] > keys add github_api YOUR_TOKEN
```

## Database queries

```bash
[recon-ng] > show hosts
[recon-ng] > show contacts
[recon-ng] > show credentials
[recon-ng] > show domains
[recon-ng] > db query SELECT * FROM hosts WHERE ip_address IS NOT NULL
```

## See also

- [[10 - theHarvester]] — simpler, single-command OSINT
- [[12 - Spiderfoot]] — GUI-based automated OSINT
