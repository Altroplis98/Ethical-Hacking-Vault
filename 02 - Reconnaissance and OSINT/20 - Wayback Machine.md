---
tags: [pentest, recon, osint, wayback, archive, both, web]
phase: 1
---
# Wayback Machine

The Internet Archive's Wayback Machine stores historical snapshots of web pages. Reveals old endpoints, removed pages, leaked info, and technology changes.

[[02 - Reconnaissance and OSINT/00 - README|Folder index]]

## Web interface

`https://web.archive.org/web/*/example.com`

## API queries

```bash
# Get all archived URLs for a domain
curl -s "https://web.archive.org/cdx/search/cdx?url=*.example.com/*&output=json&fl=original&collapse=urlkey" | \
  jq -r '.[][]' | sort -u > wayback_urls.txt

# Filter for interesting file types
grep -iE '\.(php|asp|aspx|jsp|json|xml|conf|bak|sql|env|log|old|zip|tar|gz)' wayback_urls.txt
```

## Intelligence value

| What to look for | Why |
| --- | --- |
| Old admin panels | May still be live but unlinked |
| Removed pages with sensitive content | Credentials, API docs, internal references |
| Technology stack changes | Old CMS/framework versions may still be running |
| robots.txt history | Reveals directories they tried to hide |
| JavaScript files | Old JS may contain API endpoints and keys |

## Tools that use Wayback data

- **gau** — fetches known URLs from Wayback + other sources
- **katana** — web crawler that can include Wayback data
- **waybackurls** — dedicated Wayback URL extractor

## See also

- [[21 - gau and katana]] — automated URL discovery using Wayback
- [[14 - Google Dorking]] — another way to find cached/old content
