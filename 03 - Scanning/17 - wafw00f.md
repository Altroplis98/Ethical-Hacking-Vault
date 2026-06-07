---
tags: [pentest, scanning, wafw00f, waf, firewall, both, recon, web]
tool: wafw00f
phase: 2
---
# wafw00f

Web Application Firewall (WAF) detection tool. Identifies if a target is behind a WAF and which product it is.

[[03 - Scanning/00 - README|Folder index]]

## Install / verify

```bash
which wafw00f || sudo apt install wafw00f -y
```

## Usage

```bash
# Detect WAF
wafw00f http://example.com

# Test all WAF signatures (not just first match)
wafw00f -a http://example.com

# List all detectable WAFs
wafw00f -l

# From a file
wafw00f -i targets.txt

# Output
wafw00f http://example.com -o output.json -f json
```

## Why detect WAFs

| WAF detected | Impact on testing |
| --- | --- |
| Cloudflare | Need to find origin IP to bypass |
| AWS WAF | Payload encoding may bypass rules |
| ModSecurity | Test rule set version for known bypasses |
| Akamai | Very aggressive — may block your IP |
| None detected | Direct access — standard payloads work |

## Bypassing WAFs (high level)

- Find the origin IP (DNS history, CT logs, email headers)
- Payload encoding (URL encoding, Unicode, case switching)
- HTTP parameter pollution
- Chunked transfer encoding
- Test during off-hours (some WAFs have relaxed rules)

## See also

- [[16 - WhatWeb]] — broader web fingerprinting
- [[18 - httpx]] — HTTP probing at scale
