---
tags: [pentest, enumeration, web, nikto, scanner, recon]
tool: nikto
phase: 3
---
# Nikto

Web server scanner that checks for dangerous files, outdated software, misconfigurations, and known vulnerabilities. Noisy but thorough.

[[04 - Enumeration/00 - README|Folder index]]

## Install / verify

```bash
which nikto || sudo apt install nikto -y
```

## Usage

```bash
# Basic scan
nikto -h http://10.10.10.10

# Specific port
nikto -h http://10.10.10.10:8080

# HTTPS
nikto -h https://10.10.10.10 -ssl

# Save output
nikto -h http://10.10.10.10 -o results.html -Format html
nikto -h http://10.10.10.10 -o results.txt -Format txt

# With authentication
nikto -h http://10.10.10.10 -id user:pass

# Tuning — specific test categories
nikto -h http://10.10.10.10 -Tuning 1234
```

## Tuning options

| Number | Category |
| --- | --- |
| 0 | File upload |
| 1 | Interesting file |
| 2 | Misconfiguration |
| 3 | Information disclosure |
| 4 | Injection (XSS, Script) |
| 5 | Remote file retrieval (inside web root) |
| 6 | Denial of service |
| 7 | Remote file retrieval (server-wide) |
| 8 | Command execution |
| 9 | SQL injection |

> [!warning] Nikto is noisy
> It sends thousands of requests and is easily detected by IDS/WAF. Use it only when stealth is not a concern.

## See also

- [[24 - Gobuster]] — directory brute-forcing
- [[28 - WPScan]] — CMS-specific scanning
