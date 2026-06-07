---
tags: [pentest, recon, osint, google, dorking, both]
phase: 1
---
# Google Dorking

Using advanced Google search operators to find exposed files, login pages, error messages, and sensitive data indexed by Google.

[[02 - Reconnaissance and OSINT/00 - README|Folder index]]

## Core operators

| Operator | Purpose | Example |
| --- | --- | --- |
| `site:` | Limit to specific domain | `site:example.com` |
| `inurl:` | URL must contain string | `inurl:admin` |
| `intitle:` | Page title must contain | `intitle:"index of"` |
| `filetype:` | Specific file extension | `filetype:pdf` |
| `ext:` | Same as filetype | `ext:sql` |
| `intext:` | Page body must contain | `intext:"password"` |
| `cache:` | Google's cached version | `cache:example.com` |
| `-` | Exclude term | `site:example.com -www` |
| `""` | Exact phrase match | `"default password"` |
| `*` | Wildcard | `"admin * password"` |
| `OR` / `|` | Boolean OR | `filetype:sql OR filetype:bak` |

## High-value dorks for pentesting

### Exposed files and directories

```text
site:example.com intitle:"index of"
site:example.com filetype:sql
site:example.com filetype:bak
site:example.com filetype:log
site:example.com filetype:conf OR filetype:cfg OR filetype:ini
site:example.com filetype:env
site:example.com filetype:xml intext:"password"
```

### Login pages and admin panels

```text
site:example.com inurl:admin
site:example.com inurl:login
site:example.com intitle:"dashboard"
site:example.com inurl:wp-admin
site:example.com inurl:phpmyadmin
```

### Error messages (version/stack disclosure)

```text
site:example.com "fatal error" OR "syntax error" OR "stack trace"
site:example.com "mysql_connect" OR "ORA-" OR "PostgreSQL"
site:example.com "server at" intitle:apache
```

### Sensitive data exposure

```text
site:example.com filetype:xlsx OR filetype:csv intext:"password"
site:example.com filetype:pdf "confidential"
site:example.com intext:"BEGIN RSA PRIVATE KEY"
```

## Google Hacking Database (GHDB)

The Exploit-DB maintains a curated list of dorks:
`https://www.exploit-db.com/google-hacking-database`

Categories: Footholds, Files Containing Usernames, Sensitive Directories, Web Server Detection, Vulnerable Files, Error Messages, etc.

## Automation

```bash
# Use theHarvester with Google source
theHarvester -d example.com -b google

# Manual automation (be careful of rate limits)
# Google WILL block your IP if you automate too aggressively
```

> [!warning] Google rate limits
> Automated Google queries will trigger CAPTCHAs and IP blocks. Use a VPN, slow down, or use API-based tools instead.

## See also

- [[15 - Shodan]] — search engine for exposed services (not web pages)
- [[16 - Censys]] — similar to Shodan
- [[10 - theHarvester]] — automates some Google dorking
