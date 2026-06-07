---
tags: [pentest, recon, osint, subdomain, amass, both]
tool: amass
phase: 1
---
# Amass

OWASP's comprehensive attack surface mapping tool. More thorough than subfinder but slower and noisier in active mode.

[[02 - Reconnaissance and OSINT/00 - README|Folder index]]

## Install / verify

```bash
# Kali
sudo apt install amass -y

# Or Go install
go install -v github.com/owasp-amass/amass/v4/...@master

amass -version
```

## Modes

| Mode | Stealth | Speed | Depth |
| --- | --- | --- | --- |
| `amass enum -passive` | Passive only — no target contact | Fast | Good |
| `amass enum` (default) | Active DNS resolution | Medium | Better |
| `amass enum -active` | Active + cert grabbing + zone transfers | Slow | Best |
| `amass intel` | Org/ASN/CIDR discovery | Fast | Broad |

## Core commands

```bash
# Passive enum (safe for any engagement)
amass enum -passive -d example.com -o amass_passive.txt

# Active enum (resolves found subdomains)
amass enum -d example.com -o amass_active.txt

# Active + brute force
amass enum -active -brute -d example.com -o amass_brute.txt

# Intel mode — discover domains owned by an org
amass intel -org "Example Inc"
amass intel -asn 12345
amass intel -cidr 203.0.113.0/24
```

## Key flags

| Flag | Purpose |
| --- | --- |
| `-d domain` | Target domain |
| `-passive` | Passive sources only |
| `-active` | Include active techniques (cert grabbing) |
| `-brute` | DNS brute-forcing |
| `-w wordlist` | Custom brute-force wordlist |
| `-rf resolvers.txt` | Custom DNS resolvers |
| `-bl blacklist.txt` | Exclude these subdomains |
| `-max-dns-queries N` | Rate limit |
| `-o file` | Output file |
| `-json file` | JSON output |
| `-dir dir` | Output directory for graph DB |

## Config file

`~/.config/amass/config.ini` — add API keys and data sources:

```ini
[scope]
port = 80,443,8080,8443

[data_sources.SecurityTrails]
[data_sources.SecurityTrails.Credentials]
apikey = your_key_here

[data_sources.Shodan]
[data_sources.Shodan.Credentials]
apikey = your_key_here
```

## Amass database

```bash
# Track results across multiple runs
amass db -show -d example.com

# List all domains in the DB
amass db -names -d example.com

# Compare two enum runs
amass track -d example.com
```

## See also

- [[04 - Subfinder]] — faster, simpler alternative
- [[06 - Assetfinder]] — lightweight option
- [[15 - Shodan]] — one of amass's data sources
