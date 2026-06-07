---
tags: [pentest, xss, xsstrike, web, initial-access]
tool: xsstrike
phase: 5
---
# XSStrike

Smart XSS scanner that fingerprints filters and crafts payloads to bypass them.

[[06 - Gaining Access/00 - README|Folder index]]

## Basic

```bash
xsstrike -u "https://target/?q=test"

# POST
xsstrike -u "https://target/login" --data "name=test&msg=test" --params

# With cookies
xsstrike -u "https://target/profile" --headers "Cookie: PHPSESSID=abc"

# Crawl + scan
xsstrike -u "https://target" --crawl

# Custom DOM XSS check
xsstrike -u "https://target/page?q=test" --dom
```

## When to use vs manual

- Reflected XSS in obvious params → XSStrike is fast.
- Stored XSS → manual + Burp Repeater.
- DOM XSS → browser DevTools + manual.
- Filtered / WAF'd → XSStrike's filter detection is its strength.

## See also

- [[33 - XSS Payloads]] - manual payloads to try first
