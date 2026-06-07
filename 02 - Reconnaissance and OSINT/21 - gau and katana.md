---
tags: [pentest, recon, osint, gau, katana, urls, both, web]
tool: gau, katana
phase: 1
---
# gau and katana

URL discovery tools. gau pulls historical URLs from passive sources; katana actively crawls. Together they build a comprehensive URL map.

[[02 - Reconnaissance and OSINT/00 - README|Folder index]]

## gau (Get All URLs)

Fetches known URLs from Wayback Machine, Common Crawl, AlienVault OTX, and URLScan.

```bash
# Install
go install github.com/lc/gau/v2/cmd/gau@latest

# Basic usage
echo "example.com" | gau

# With filters
echo "example.com" | gau --threads 5 --o urls.txt

# Filter by extension
echo "example.com" | gau | grep -iE '\.(js|json|xml|php|asp|aspx|env|bak|sql)$'

# Filter by status code (if paired with httpx)
echo "example.com" | gau | httpx -silent -status-code -mc 200
```

## katana

Active web crawler by ProjectDiscovery. Crawls a target and discovers endpoints, forms, and JavaScript files.

```bash
# Install
go install github.com/projectdiscovery/katana/cmd/katana@latest

# Basic crawl
katana -u https://example.com

# Crawl with depth and output
katana -u https://example.com -d 3 -o crawled.txt

# Crawl from a list
katana -list targets.txt -d 2 -o all_crawled.txt

# JavaScript parsing (extract endpoints from JS files)
katana -u https://example.com -jc -d 2

# Headless browser mode (renders JavaScript)
katana -u https://example.com -headless -d 2
```

### Key katana flags

| Flag | Purpose |
| --- | --- |
| `-u url` | Target URL |
| `-list file` | File of target URLs |
| `-d N` | Crawl depth (default 3) |
| `-jc` | JavaScript crawl — parse JS files for endpoints |
| `-headless` | Use headless browser (Chrome) |
| `-o file` | Output file |
| `-silent` | Clean output |
| `-ef ext` | Exclude file extensions |
| `-f ext` | Filter to specific extensions |

## Combined workflow

```bash
# Passive (gau) + Active (katana) → deduplicate
echo "example.com" | gau > urls_passive.txt
katana -u https://example.com -d 3 -jc > urls_active.txt
cat urls_passive.txt urls_active.txt | sort -u > all_urls.txt

# Find interesting endpoints
grep -iE '(api|admin|dashboard|login|upload|config|internal)' all_urls.txt
```

## See also

- [[20 - Wayback Machine]] — gau's primary data source
- [[22 - Gitleaks and TruffleHog]] — scan discovered URLs/repos for secrets
