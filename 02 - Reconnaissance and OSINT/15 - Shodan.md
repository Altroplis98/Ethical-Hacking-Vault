---
tags: [pentest, recon, osint, shodan, both]
tool: shodan
phase: 1
---
# Shodan

Search engine for internet-connected devices. Indexes banners, certificates, and services — shows you what's exposed before you scan a single port.

[[02 - Reconnaissance and OSINT/00 - README|Folder index]]

## Install CLI

```bash
pip install shodan --break-system-packages
shodan init YOUR_API_KEY
```

## Web interface

`https://www.shodan.io` — free account gives limited results; paid membership unlocks filters and bulk queries.

## CLI usage

```bash
# Search for a target
shodan search "hostname:example.com"

# Host info
shodan host 203.0.113.10

# Count results
shodan count "apache country:US"

# Download results (paid)
shodan download results "org:\"Example Inc\""
shodan parse results.json.gz --fields ip_str,port,org

# Scan a target (uses your Shodan credits)
shodan scan submit 203.0.113.0/24
```

## Key search filters

| Filter | Example | Purpose |
| --- | --- | --- |
| `hostname:` | `hostname:example.com` | Target domain |
| `ip:` | `ip:203.0.113.10` | Specific IP |
| `net:` | `net:203.0.113.0/24` | CIDR range |
| `org:` | `org:"Example Inc"` | Organization name |
| `port:` | `port:3389` | Specific port open |
| `product:` | `product:Apache` | Specific software |
| `version:` | `version:2.4.49` | Specific version |
| `os:` | `os:"Windows Server 2019"` | Operating system |
| `country:` | `country:US` | Country code |
| `city:` | `city:Seattle` | City |
| `ssl.cert.subject.cn:` | `ssl.cert.subject.cn:example.com` | SSL cert common name |
| `vuln:` | `vuln:CVE-2021-44228` | Known vulnerability (paid) |
| `has_screenshot:true` | — | Devices with screenshots |

## High-value dork examples

```text
# Exposed RDP
port:3389 "Authentication: disabled" org:"Example Inc"

# Default credentials on webcams
"Server: yawcam" "Mime-Type: text/html"

# Exposed MongoDB
port:27017 "MongoDB Server Information" org:"Example Inc"

# Exposed Elasticsearch
port:9200 "elasticsearch" org:"Example Inc"

# Jenkins without auth
"X-Jenkins" "Set-Cookie: JSESSIONID" http.title:"Dashboard"
```

## API usage (Python)

```python
import shodan
api = shodan.Shodan('YOUR_API_KEY')

# Search
results = api.search('hostname:example.com')
for r in results['matches']:
    print(f"{r['ip_str']}:{r['port']} - {r.get('product','unknown')}")

# Host lookup
host = api.host('203.0.113.10')
print(f"Ports: {host['ports']}")
print(f"Vulns: {host.get('vulns', [])}")
```

## See also

- [[16 - Censys]] — similar service, different index
- [[14 - Google Dorking]] — web content search vs Shodan's service/banner search
