---
tags: [pentest, enumeration, web, joomla, joomscan, recon]
tool: joomscan
phase: 3
---
# JoomScan

OWASP Joomla vulnerability scanner. Detects Joomla version, components, and known vulnerabilities.

[[04 - Enumeration/00 - README|Folder index]]

## Install / verify

```bash
which joomscan || sudo apt install joomscan -y
```

## Usage

```bash
# Basic scan
joomscan -u http://10.10.10.10

# Enumerate components
joomscan -u http://10.10.10.10 -ec

# Custom User-Agent
joomscan -u http://10.10.10.10 -a "Mozilla/5.0"

# Set cookie
joomscan -u http://10.10.10.10 --cookie "session=abc123"
```

## What it checks

- Joomla version detection
- Admin page discovery
- Known vulnerabilities for detected version
- Component enumeration
- Configuration file exposure
- Directory listing

## See also

- [[28 - WPScan]] — WordPress equivalent
- [[30 - Droopescan]] — Drupal scanner
