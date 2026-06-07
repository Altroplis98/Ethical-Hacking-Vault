---
tags: [pentest, enumeration, web, dirsearch, directory, recon]
tool: dirsearch
phase: 3
---
# dirsearch

Python-based web path scanner with a good built-in wordlist and recursive scanning.

[[04 - Enumeration/00 - README|Folder index]]

## Install / verify

```bash
which dirsearch || pip install dirsearch --break-system-packages
```

## Usage

```bash
# Basic scan (uses built-in wordlist)
dirsearch -u http://10.10.10.10

# Custom wordlist
dirsearch -u http://10.10.10.10 -w /usr/share/wordlists/dirb/common.txt

# Extensions
dirsearch -u http://10.10.10.10 -e php,html,txt

# Recursive
dirsearch -u http://10.10.10.10 -r

# Exclude status codes
dirsearch -u http://10.10.10.10 -x 404,403

# Output
dirsearch -u http://10.10.10.10 -o results.txt --format json
```

## See also

- [[24 - Gobuster]] — faster Go-based alternative
- [[26 - Feroxbuster]] — recursive Rust-based option
