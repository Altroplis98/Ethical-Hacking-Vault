---
tags: [pentest, cracking, john, jtr, both, initial-access]
tool: john
phase: 7
---
# John the Ripper Reference

Classic password cracker. CPU-based (slower than GPU hashcat) but handles more exotic formats and has excellent *2john extraction tools.

[[00 - README|Folder index]]

## Install / verify

```bash
which john
sudo apt install john
# Jumbo version (community) has more formats:
sudo apt install john-the-ripper
```

## Basic usage

```bash
# Auto-detect hash type and crack
john hashes.txt

# Specify wordlist
john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt

# Specify format
john --format=NT hashes.txt

# Show cracked passwords
john --show hashes.txt
```

## Common formats

| `--format=` | Hash type |
| --- | --- |
| `Raw-MD5` | Plain MD5 |
| `Raw-SHA256` | Plain SHA-256 |
| `NT` | NTLM (Windows) |
| `sha512crypt` | Linux shadow `$6$` |
| `bcrypt` | bcrypt `$2*$` |
| `krb5tgs` | Kerberoast |
| `krb5asrep` | AS-REP Roast |
| `netntlmv2` | NetNTLMv2 |

## *2john extraction tools

John's killer feature — convert files to crackable hashes:

```bash
ssh2john id_rsa > id_rsa.hash          # SSH private key passphrase
zip2john protected.zip > zip.hash      # ZIP password
rar2john protected.rar > rar.hash      # RAR password
pdf2john protected.pdf > pdf.hash      # PDF password
office2john document.docx > doc.hash   # Office password
keepass2john database.kdbx > kp.hash   # KeePass master password
bitlocker2john drive.img > bl.hash     # BitLocker recovery
gpg2john private.gpg > gpg.hash        # GPG passphrase

# Then crack:
john --wordlist=rockyou.txt zip.hash

AlexAviles@htb[/htb]$ bitlocker2john -i Backup.vhd > backup.hashes
AlexAviles@htb[/htb]$ grep "bitlocker\$0" backup.hashes > backup.hash
AlexAviles@htb[/htb]$ cat backup.hash
```


## Rules

```bash
# Use built-in rules
john --wordlist=rockyou.txt --rules hashes.txt

# Use specific rule section
john --wordlist=rockyou.txt --rules=KoreLogic hashes.txt
```

## Session management

```bash
# Name a session
john --session=mycrack --wordlist=rockyou.txt hashes.txt

# Restore interrupted session
john --restore=mycrack

# Status
john --status=mycrack
```

> [!tip] John for extraction, hashcat for cracking
> Use John's `*2john` tools to extract hashes from files, then crack with hashcat for GPU speed. Best of both worlds.

## See also

- [[20 - Hashcat Mode IDs]]
- [[25 - Archive Cracking (zip 7z rar pdf office)]]
- [[26 - Hash Identification]]
