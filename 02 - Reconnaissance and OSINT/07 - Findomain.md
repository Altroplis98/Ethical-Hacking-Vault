---
tags: [pentest, recon, osint, subdomain, findomain, both]
tool: findomain
phase: 1
---
# Findomain

Cross-platform subdomain finder with monitoring capabilities. Fast, written in Rust.

[[02 - Reconnaissance and OSINT/00 - README|Folder index]]

## Install

```bash
# Download latest release
curl -LO https://github.com/Findomain/Findomain/releases/latest/download/findomain-linux.zip
unzip findomain-linux.zip
chmod +x findomain
sudo mv findomain /usr/local/bin/
```

## Usage

```bash
# Basic enumeration
findomain -t example.com

# Output to file
findomain -t example.com -u subs.txt

# Multiple targets
findomain -f domains.txt -u all_subs.txt

# Resolve found subdomains
findomain -t example.com -r

# JSON output
findomain -t example.com --json output.json
```

## Key flags

| Flag | Purpose |
| --- | --- |
| `-t domain` | Target domain |
| `-f file` | File with target domains |
| `-u file` | Unique output file |
| `-r` | Resolve subdomains |
| `--json file` | JSON output |
| `-q` | Quiet mode |

## Monitoring mode

Findomain can detect new subdomains over time — useful for red team persistence:

```bash
findomain -t example.com --monitoring-flag
```

## See also

- [[04 - Subfinder]] — most popular alternative
- [[08 - Sublist3r]] — Python-based option
