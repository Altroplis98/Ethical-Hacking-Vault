---
tags: [pentest, scanning, whatweb, fingerprint, web, both, recon]
tool: whatweb
phase: 2
---
# WhatWeb

Web technology fingerprinter. Identifies CMS, frameworks, web servers, JavaScript libraries, analytics platforms, and more.

[[03 - Scanning/00 - README|Folder index]]

## Install / verify

```bash
which whatweb || sudo apt install whatweb -y
```

## Usage

```bash
# Basic fingerprint
whatweb http://10.10.10.10

# Aggressive mode (more requests, more info)
whatweb -a 3 http://10.10.10.10

# Multiple targets
whatweb -i targets.txt

# Verbose JSON output
whatweb --log-json=output.json http://10.10.10.10

# Follow redirects
whatweb -r http://10.10.10.10
```

## Aggression levels

| Level | Description |
| --- | --- |
| 1 (Stealthy) | One HTTP request. Default. |
| 3 (Aggressive) | Multiple requests, tries common paths |
| 4 (Heavy) | Many requests, brute-force paths |

## What it detects

CMS (WordPress, Joomla, Drupal), web servers (Apache, Nginx, IIS), frameworks (Rails, Django, ASP.NET), JavaScript (jQuery, React, Angular), email addresses, IP addresses, HTTP headers, cookies, and 1800+ plugins.

## See also

- [[17 - wafw00f]] — WAF detection
- [[18 - httpx]] — bulk HTTP probing
