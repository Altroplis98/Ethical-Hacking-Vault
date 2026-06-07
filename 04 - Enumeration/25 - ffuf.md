---
tags: [pentest, enumeration, web, ffuf, fuzzing, recon]
tool: ffuf
phase: 3
---
# ffuf

Fast web fuzzer. More flexible than gobuster — fuzzes any part of the request (URL, headers, POST body, cookies).

[[04 - Enumeration/00 - README|Folder index]]

## Install / verify

```bash
which ffuf || sudo apt install ffuf -y
```

## Directory brute-force

```bash
ffuf -u http://10.10.10.10/FUZZ -w /usr/share/seclists/Discovery/Web-Content/common.txt

# With extensions
ffuf -u http://10.10.10.10/FUZZ -w /usr/share/seclists/Discovery/Web-Content/common.txt -e .php,.html,.txt
```

## Vhost / subdomain discovery

```bash
# Vhost fuzzing
ffuf -u http://10.10.10.10 -H "Host: FUZZ.example.com" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -fs 612

# -fs filters out responses with the default page size (false positives)
```

## POST parameter fuzzing

```bash
# Fuzz password field
ffuf -u http://10.10.10.10/login -X POST -d "username=admin&password=FUZZ" -w passwords.txt -H "Content-Type: application/x-www-form-urlencoded"

# Fuzz both fields
ffuf -u http://10.10.10.10/login -X POST -d "username=UFUZZ&password=PFUZZ" -w users.txt:UFUZZ -w passwords.txt:PFUZZ -H "Content-Type: application/x-www-form-urlencoded" -mode clusterbomb
```

## Filtering results

| Flag | Filter by |
| --- | --- |
| `-fc N` | Filter by status code |
| `-fs N` | Filter by response size |
| `-fw N` | Filter by word count |
| `-fl N` | Filter by line count |
| `-mc N` | Match by status code |
| `-ms N` | Match by response size |

## Key flags

| Flag | Purpose |
| --- | --- |
| `-u url` | Target URL (FUZZ = placeholder) |
| `-w wordlist` | Wordlist (wordlist:NAME for multiple) |
| `-X method` | HTTP method |
| `-d data` | POST data |
| `-H header` | Custom header |
| `-b cookie` | Cookie |
| `-t N` | Threads (default 40) |
| `-o file` | Output file |
| `-of format` | Output format (json, csv, html) |
| `-r` | Follow redirects |
| `-mode` | clusterbomb or pitchfork |
| `-e exts` | Extensions |

## See also

- [[24 - Gobuster]] — simpler directory brute-forcing
- [[26 - Feroxbuster]] — recursive discovery
