---
tags: [pentest, scanning, httpx, web, http, both, recon]
tool: httpx
phase: 2
---
# httpx

Fast HTTP toolkit by ProjectDiscovery. Probes live web servers, grabs titles, status codes, technologies, and more. Essential pipeline tool.

[[03 - Scanning/00 - README|Folder index]]

## Install

```bash
go install -v github.com/projectdiscovery/httpx/cmd/httpx@latest
```

## Usage

```bash
# Probe a list of hosts/URLs
cat hosts.txt | httpx

# With details
cat hosts.txt | httpx -title -status-code -tech-detect -content-length

# From subfinder
subfinder -d example.com -silent | httpx -silent

# Single target, all info
echo "10.10.10.10" | httpx -title -status-code -content-length -tech-detect -follow-redirects

# Screenshot (saves PNG)
cat hosts.txt | httpx -screenshot
```

## Key flags

| Flag | Purpose |
| --- | --- |
| `-title` | Show page title |
| `-status-code` / `-sc` | HTTP status code |
| `-content-length` / `-cl` | Response body size |
| `-tech-detect` / `-td` | Technology detection (Wappalyzer) |
| `-follow-redirects` / `-fr` | Follow HTTP redirects |
| `-silent` | Clean output (URLs only) |
| `-o file` | Output file |
| `-json` | JSON output |
| `-screenshot` | Take screenshots |
| `-probe` | Show probe status |
| `-web-server` | Web server header |
| `-method` | HTTP method |
| `-mc codes` | Match specific status codes |
| `-fc codes` | Filter out specific status codes |

## Pipeline recipes

```bash
# Find all live web servers from nmap output
grep -oP '\d+\.\d+\.\d+\.\d+' nmap_scan.gnmap | sort -u | httpx -silent

# Screenshot all discovered web apps
subfinder -d example.com -silent | httpx -silent | httpx -screenshot -o screenshots/

# Find login pages
cat hosts.txt | httpx -title -silent | grep -iE 'login|sign.in|admin'
```

## See also

- [[16 - WhatWeb]] — deeper single-target fingerprinting
- [[19 - Nuclei Tech Detection]] — vulnerability scanning after httpx
