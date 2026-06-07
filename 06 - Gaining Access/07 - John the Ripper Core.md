---
tags: [pentest, john, password-cracking, hashes, initial-access]
tool: john
phase: 5
---
# John the Ripper - Core

CPU-focused password cracker. Strengths: format auto-detection, archive helpers (`zip2john`, etc.), "jumbo" build supports almost every format.

[[06 - Gaining Access/00 - README|Folder index]]

## Install / verify

```bash
john --list=formats | head
john --version
which zip2john pdf2john.pl ssh2john office2john rar2john 7z2john bitcoin2john
```

## Skeleton

```bash
john [--format=X] [--wordlist=W] [--rules=R] hashfile
```

## Common modes

### Wordlist

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
john --wordlist=rockyou.txt --rules=Single hash.txt
john --wordlist=rockyou.txt --rules=KoreLogic hash.txt
john --wordlist=rockyou.txt --rules=All hash.txt
```

### Incremental (John's brute)

```bash
john --incremental hash.txt
john --incremental=Alpha hash.txt           # only letters
john --incremental=Digits hash.txt
john --incremental=ASCII hash.txt
```

### Single mode (use the username/GECOS as a guess)

```bash
john --single hash.txt
# Useful for hashes in format  user:hash  - tries username as password
```

### Show results

```bash
john --show hash.txt
john --show --format=NT hash.txt
```

## Format hint examples

```bash
john --format=raw-md5
john --format=Raw-SHA1
john --format=nt                # NTLM
john --format=sha512crypt       # Linux $6$
john --format=md5crypt          # Linux $1$
john --format=krb5tgs           # Kerberoast
john --format=krb5asrep         # AS-REP roast
john --format=netntlmv2         # Responder captures
john --format=zip
john --format=7z
john --format=pdf
john --format=office
john --format=ssh               # encrypted private keys
```

## Helpers - turn artifacts into john-friendly hashes

```bash
ssh2john id_rsa > id_rsa.john
john --wordlist=rockyou.txt id_rsa.john

zip2john secret.zip > zip.john
john --wordlist=rockyou.txt zip.john

7z2john archive.7z > 7z.john
rar2john file.rar > rar.john
office2john document.docx > off.john
office2john.py Protected.docx > protected-docx.hash
pdf2john.pl file.pdf > pdf.john
pdf2john.py PDF.pdf > pdf.hash
bitcoin2john wallet.dat > btc.john
keepass2john file.kdbx > kp.john
hashcat-cli on /etc/shadow:
unshadow /etc/passwd /etc/shadow > combined.txt    # then john combined.txt
```

## Hashcat ↔ John equivalences

| Hashcat -m | John --format= |
| ---: | --- |
| 0 | raw-md5 |
| 100 | raw-sha1 |
| 1000 | NT |
| 5500 | netntlm |
| 5600 | netntlmv2 |
| 13100 | krb5tgs |
| 18200 | krb5asrep |
| 1800 | sha512crypt |
| 500 | md5crypt |
| 1700 | raw-sha512 |
| 9600 | office13 |
| 13600 | zip |

## When to pick John over Hashcat

| Need | Pick |
| --- | --- |
| GPU available, generic hash | Hashcat |
| Quick check + you only have CPU | John |
| Archive helper (`pdf2john`, `zip2john`) | John (you'll use the helper, then move to Hashcat for big runs) |
| Format auto-detect needed | John |
| Complex rule engine on a small list | John (incremental + Single + rules combo) |

> [!tip] Hybrid workflow
> Most pros use BOTH. `zip2john` to produce the hash → `hashcat -m 13600` for big GPU runs. Or `john --single` first (instant, sometimes hits username-as-password) → then hashcat with rules.
