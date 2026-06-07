---
tags: [pentest, recon, osint, subdomain, assetfinder, both]
tool: assetfinder
phase: 1
---
# Assetfinder

Minimal, fast subdomain finder by Tomnomnom. No config files, no API keys needed. Good for quick sweeps.

[[02 - Reconnaissance and OSINT/00 - README|Folder index]]

## Install

```bash
go install github.com/tomnomnom/assetfinder@latest
```

## Usage

```bash
# Find subdomains
assetfinder example.com

# Only subdomains of the target (exclude related domains)
assetfinder --subs-only example.com

# Pipe to further tools
assetfinder --subs-only example.com | sort -u | httpx -silent
```

## When to use assetfinder vs. subfinder vs. amass

| Tool | Speed | Depth | Config needed |
| --- | --- | --- | --- |
| assetfinder | Fastest | Shallow | None |
| subfinder | Fast | Good | API keys optional |
| amass | Slow | Deep | Config recommended |

Use assetfinder for a quick first pass, then subfinder or amass for thoroughness.

## See also

- [[04 - Subfinder]] — more sources, still fast
- [[05 - Amass]] — most thorough
- [[07 - Findomain]] — another fast alternative
