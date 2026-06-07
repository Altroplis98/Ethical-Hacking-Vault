---
tags: [pentest, enumeration, dns, fierce, recon]
tool: fierce
phase: 3
---
# fierce

Lightweight DNS reconnaissance scanner. Discovers adjacent IP space and non-contiguous networks.

[[04 - Enumeration/00 - README|Folder index]]

## Usage

```bash
fierce --domain example.com
fierce --domain example.com --subdomain-file /path/to/wordlist.txt
fierce --domain example.com --dns-servers 8.8.8.8
fierce --domain example.com --wide  # expand to nearby IPs
```

> Full details in [[02 - Reconnaissance and OSINT/09 - dnsenum dnsrecon fierce|dnsenum dnsrecon fierce]].

## See also

- [[18 - dnsenum]] — broader DNS enum
- [[19 - dnsrecon]] — most flexible DNS tool
