---
tags: [pentest, recon, osint, certificates, ct, both]
phase: 1
---
# Certificate Transparency

CT logs are public records of every SSL/TLS certificate issued. They reveal subdomains, internal hostnames, and org structure that the target may not want public.

[[02 - Reconnaissance and OSINT/00 - README|Folder index]]

## Why CT logs matter

When a CA issues a cert for `internal-vpn.corp.example.com`, that name goes into a public CT log. The target may not realize these internal names are exposed.

## Query methods

### crt.sh (free, web + API)

```bash
# Web: https://crt.sh/?q=%25.example.com

# API query
curl -s "https://crt.sh/?q=%25.example.com&output=json" | \
  jq -r '.[].name_value' | sort -u

# Include expired certs (may reveal old infrastructure)
curl -s "https://crt.sh/?q=%25.example.com&output=json&deduplicate=Y" | \
  jq -r '.[].name_value' | sort -u
```

### Other CT search engines

| Tool | URL | Notes |
| --- | --- | --- |
| crt.sh | crt.sh | Free, API available |
| Censys | search.censys.io | Richer metadata, requires account |
| Google CT | transparencyreport.google.com/https/certificates | Google's own CT viewer |
| Facebook CT | developers.facebook.com/tools/ct | Monitoring tool |

## What to look for

| Pattern | Intelligence value |
| --- | --- |
| `vpn.example.com`, `remote.example.com` | VPN endpoints — high-value targets |
| `staging.example.com`, `dev.example.com` | Dev/staging environments — often less hardened |
| `jenkins.example.com`, `gitlab.example.com` | CI/CD — potential cred stores |
| `*.internal.example.com` | Internal naming conventions revealed |
| Wildcard certs (`*.example.com`) | Shows they use wildcards — subdomain takeover risk |
| Old/expired certs | Historical infrastructure, decommissioned but maybe still reachable |

## Automation with subfinder

```bash
# Subfinder uses CT logs as one of its passive sources
subfinder -d example.com -silent | sort -u
```

## Monitoring for new certs

```bash
# Set up a crt.sh RSS feed or use certspotter
# Useful for red team ops — detect when target issues new certs
```

## See also

- [[01 - WHOIS]] — pair CT findings with WHOIS ownership data
- [[04 - Subfinder]] — aggregates CT + many other passive sources
- [[05 - Amass]] — most comprehensive passive enumeration
