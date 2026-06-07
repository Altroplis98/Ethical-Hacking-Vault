---
tags: [pentest, recon, osint, censys, both]
tool: censys
phase: 1
---
# Censys

Internet-wide scanning data — hosts, certificates, and services. Complementary to Shodan with different scanning infrastructure and data format.

[[02 - Reconnaissance and OSINT/00 - README|Folder index]]

## Install CLI

```bash
pip install censys --break-system-packages
censys config  # enter API ID and Secret
```

## Web interface

`https://search.censys.io` — free tier gives 250 queries/month.

## CLI usage

```bash
# Search hosts
censys search "services.http.response.html_title: 'Example'" --index-type hosts

# View specific host
censys view 203.0.113.10 --index-type hosts

# Search certificates
censys search "parsed.subject.common_name: example.com" --index-type certificates
```

## Search query syntax

```text
# By IP
ip: 203.0.113.10

# By domain in cert
services.tls.certificates.leaf.names: example.com

# By service
services.service_name: SSH AND services.port: 22

# By HTTP title
services.http.response.html_title: "Login"

# By organization (ASN)
autonomous_system.name: "Example Inc"

# By software
services.software.product: "Apache" AND services.software.version: "2.4.49"
```

## Censys vs. Shodan

| Feature | Censys | Shodan |
| --- | --- | --- |
| Scan frequency | Continuous | Continuous |
| Certificate search | Excellent | Good |
| Banner data | Good | Excellent |
| Historical data | Limited free | Limited free |
| Vulnerability tagging | By CVE | By CVE (paid) |
| API free tier | 250 queries/month | 100 queries/month |

> [!tip] Use both
> They scan from different infrastructure and index differently. A host missing from Shodan might appear in Censys and vice versa.

## See also

- [[15 - Shodan]] — the other major internet scanning search engine
- [[03 - Certificate Transparency]] — free cert discovery
