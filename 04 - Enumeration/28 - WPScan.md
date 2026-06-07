---
tags: [pentest, enumeration, web, wordpress, wpscan, recon]
tool: wpscan
phase: 3
---
# WPScan

WordPress-specific vulnerability scanner. Enumerates users, plugins, themes, and checks for known vulns.

[[04 - Enumeration/00 - README|Folder index]]

## Install / verify

```bash
which wpscan || sudo apt install wpscan -y
```

## Usage

```bash
# Basic scan
wpscan --url http://10.10.10.10

# Enumerate users
wpscan --url http://10.10.10.10 -e u

# Enumerate vulnerable plugins
wpscan --url http://10.10.10.10 -e vp

# Enumerate all plugins
wpscan --url http://10.10.10.10 -e ap

# Enumerate themes
wpscan --url http://10.10.10.10 -e vt

# Full enumeration
wpscan --url http://10.10.10.10 -e u,vp,vt,cb,dbe

# Password brute-force
wpscan --url http://10.10.10.10 -U admin -P /usr/share/wordlists/rockyou.txt

# With API token (for vulnerability data)
wpscan --url http://10.10.10.10 --api-token YOUR_TOKEN -e vp,u
```

## Enumeration flags

| Flag | Enumerates |
| --- | --- |
| `u` | Users |
| `vp` | Vulnerable plugins |
| `ap` | All plugins |
| `vt` | Vulnerable themes |
| `at` | All themes |
| `cb` | Config backups |
| `dbe` | Database exports |

## API token

Register at `https://wpscan.com` for a free API token (25 requests/day). Without it, you get plugin/theme names but not vulnerability data.

## See also

- [[29 - JoomScan]] — Joomla equivalent
- [[30 - Droopescan]] — Drupal/SilverStripe scanner
