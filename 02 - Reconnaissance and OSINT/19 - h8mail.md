---
tags: [pentest, recon, osint, breach, h8mail, both]
tool: h8mail
phase: 1
---
# h8mail

Email OSINT and breach hunting tool. Queries multiple breach databases to find leaked credentials associated with email addresses.

[[02 - Reconnaissance and OSINT/00 - README|Folder index]]

## Install

```bash
pip install h8mail --break-system-packages
```

## Usage

```bash
# Single email
h8mail -t target@example.com

# Multiple emails
h8mail -t emails.txt

# With API keys for premium sources
h8mail -t target@example.com -k apikeys.ini

# Output to file
h8mail -t target@example.com -o results.csv
```

## API keys config (`apikeys.ini`)

```ini
[h8mail]
haveibeenpwned = YOUR_HIBP_KEY
leak-lookup_pub = YOUR_KEY
snusbase_token = YOUR_KEY
snusbase_url = https://api.snusbase.com
```

## Data sources

| Source | Type | Cost |
| --- | --- | --- |
| HaveIBeenPwned | Breach notification | $3.50/month API |
| Snusbase | Breach search + passwords | Paid |
| Leak-Lookup | Breach search | Free tier available |
| Local breach files | Direct search | Free (you need the data) |

## Using local breach databases

```bash
# Search a local breach compilation
h8mail -t target@example.com -lb /path/to/breachcompilation/ -gz
```

## Ethical considerations

> [!danger] Legal boundaries
> Accessing and using breach data varies by jurisdiction. In authorized pentests, finding client credentials in breaches is valuable intelligence. Document your source and stay within ROE.

## See also

- [[18 - holehe]] — check which services an email is registered on
- [[17 - CrossLinked]] — generate email lists to feed into h8mail
