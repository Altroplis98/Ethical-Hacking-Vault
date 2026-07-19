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

# Cameras
title:"webcamXP" country:"US"
product:"Hikvision IP Camera"
"Server: IP Camera" org:"Comcast"

# Routers / default creds
"Default Password" product:"MikroTik"
title:"RouterOS" port:80
"D-Link" "200 OK" port:8080

# Industrial / ICS / IoT
port:502 "Modbus"
port:47808 "BACnet"
"SIMATIC" port:102

# Printers
"@PJL INFO" port:9100
title:"Printer Status" port:80

# Admin panels
http.title:"Admin Panel"
http.title:"pfSense" port:443
http.title:"Untangle" port:443

# Exposed databases
port:27017 product:"MongoDB" -authentication
port:6379 "redis_version" -auth

# VPNs / remote access
"Cisco ASA" port:443
"Pulse Secure" port:443
title:"SSL VPN" port:4433

# Targeted org recon
org:"Target" port:22
org:"Target" http.title:"Login"
org:"Target" ssl.cert.subject.cn:"targetdomain.com"
```

## OSINT recon workflow (org targeting)

```text
# 1. Find what an org has exposed
org:"Company Name"

# 2. Narrow to interesting services
org:"Company Name" port:443
org:"Company Name" http.title:"Login"
org:"Company Name" product:"Cisco"

# 3. Check for known vulns (paid)
org:"Company Name" vuln:CVE-2021-44228

# 4. Find subdomains via SSL certs
ssl.cert.subject.cn:"*.example.com"
ssl.cert.subject.cn:"example.com"

# 5. Find dev/staging environments
hostname:"dev.example.com" OR hostname:"staging.example.com"

# 6. Pivot on ASN (more accurate than org name)
# Find ASN first: https://bgp.he.net → search company name
asn:AS12345
```

## Tips

- `org:` uses Shodan's own org tagging — try variations if no results ("Example Inc", "Example", "Example LLC")
- `net:` is more reliable than `org:` if you know the IP range — pull it from whois or bgp.he.net
- Combine filters with spaces (AND) — no native OR, use two separate searches
- `has_screenshot:true` combined with `title:` is fast for finding exposed web UIs visually
- `vuln:` filter is paid-only — free alternative: cross-ref version numbers manually against NVD
- Shodan indexes at crawl time — results may be weeks old, always verify with a live scan

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
