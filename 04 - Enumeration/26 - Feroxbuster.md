---
tags: [pentest, enumeration, web, feroxbuster, directory, recon]
tool: feroxbuster
phase: 3
---
# Feroxbuster

Recursive content discovery tool written in Rust. Automatically dives into discovered directories.

[[04 - Enumeration/00 - README|Folder index]]

## Install / verify

```bash
which feroxbuster || sudo apt install feroxbuster -y
```

## Usage

```bash
# Basic recursive scan
feroxbuster -u http://10.10.10.10

# Custom wordlist
feroxbuster -u http://10.10.10.10 -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt

# With extensions
feroxbuster -u http://10.10.10.10 -x php,html,txt

# Limit recursion depth
feroxbuster -u http://10.10.10.10 -d 2

# Filter by status code
feroxbuster -u http://10.10.10.10 -C 404,403

# Filter by response size
feroxbuster -u http://10.10.10.10 -S 0

# Output
feroxbuster -u http://10.10.10.10 -o results.txt
```

## Why feroxbuster

- **Recursive by default** — automatically scans discovered directories
- **Fast** — Rust performance
- **Smart filtering** — auto-detects and filters false positives
- **Resume capability** — can resume interrupted scans

## See also

- [[24 - Gobuster]] — non-recursive, simpler
- [[25 - ffuf]] — most flexible for custom fuzzing
