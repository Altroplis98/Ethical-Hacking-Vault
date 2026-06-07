---
tags: [pentest, enumeration, web, gobuster, directory, recon]
tool: gobuster
phase: 3
---
# Gobuster

Fast directory/file brute-forcer, DNS subdomain enumerator, and vhost discoverer. Written in Go.

[[04 - Enumeration/00 - README|Folder index]]

**Directories/files:**

- `/usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt` — great balance of coverage and speed
- `/usr/share/seclists/Discovery/Web-Content/raft-large-directories.txt` — when medium misses things
- `/usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt` — solid alternative

**Subdomains/vhosts:**

- `/usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt` — fast first pass
- `/usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt` — if 5k misses
- `/usr/share/seclists/Discovery/DNS/n0kovo_subdomains.txt` — excellent and modern
## Install / verify

```bash
which gobuster || sudo apt install gobuster -y
```

## Directory / file brute-force

```bash
# Basic directory brute
gobuster dir -u http://10.10.10.10 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50 --no-error

# With extensions
gobuster dir -u http://10.10.10.10 -w /usr/share/wordlists/dirb/common.txt -x php,html,txt,bak

# Custom status codes
gobuster dir -u http://10.10.10.10 -w wordlist.txt -s 200,301,302,403

# With cookies / auth
gobuster dir -u http://10.10.10.10 -w wordlist.txt -c "session=abc123"
gobuster dir -u http://10.10.10.10 -w wordlist.txt -U user -P pass

# Follow redirects
gobuster dir -u http://10.10.10.10 -w wordlist.txt -r

# Output
gobuster dir -u http://10.10.10.10 -w wordlist.txt -o results.txt
```

## DNS subdomain brute-force

```bash
gobuster dns -d example.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -t 50
```

## Vhost discovery

```bash
gobuster vhost -u http://10.10.10.10 -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt --append-domain
```

## Key flags

| Flag | Purpose |
| --- | --- |
| `-u url` | Target URL |
| `-w wordlist` | Wordlist path |
| `-x exts` | File extensions to try |
| `-t N` | Threads (default 10) |
| `-s codes` | Show status codes |
| `-b codes` | Hide status codes |
| `-r` | Follow redirects |
| `-c cookie` | Cookie string |
| `-H header` | Custom header |
| `-o file` | Output file |
| `-k` | Skip TLS verification |

## See also

- [[25 - ffuf]] — more flexible fuzzer
- [[26 - Feroxbuster]] — recursive directory brute-forcer
