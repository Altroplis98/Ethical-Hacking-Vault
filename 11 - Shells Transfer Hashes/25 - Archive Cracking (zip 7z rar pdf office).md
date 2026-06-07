---
tags: [pentest, cracking, archive, zip, rar, pdf, office, both, initial-access]
type: cheatsheet
phase: 7
---
# Archive Cracking (zip / 7z / rar / pdf / office)

Extract password hashes from encrypted files, then crack with John or hashcat.

[[00 - README|Folder index]]

## ZIP

```bash
# Extract hash
zip2john protected.zip > zip.hash

# Crack with John
john --wordlist=rockyou.txt zip.hash

# Crack with hashcat
# Check zip2john output for hash type:
# PKZIP → hashcat -m 17200/17210/17220/17225/17230
# WinZip → hashcat -m 13600
hashcat -m 17200 zip.hash rockyou.txt
```

## RAR

```bash
# Extract hash
rar2john protected.rar > rar.hash

# Crack
john --wordlist=rockyou.txt rar.hash

# Hashcat: RAR3 = -m 12500, RAR5 = -m 13000
hashcat -m 13000 rar.hash rockyou.txt
```

## 7-Zip

```bash
# Extract hash
7z2john protected.7z > 7z.hash

# Crack
john --wordlist=rockyou.txt 7z.hash

# Hashcat: -m 11600
hashcat -m 11600 7z.hash rockyou.txt
```

## PDF

```bash
# Extract hash
pdf2john protected.pdf > pdf.hash

# Crack
john --wordlist=rockyou.txt pdf.hash

# Hashcat: PDF 1.1-1.3 = -m 10400, PDF 1.4-1.6 = -m 10500, PDF 1.7 = -m 10600/10700
```

## Microsoft Office

```bash
# Extract hash
office2john document.docx > office.hash

# Crack
john --wordlist=rockyou.txt office.hash

# Hashcat modes:
# Office 2007 = -m 9400
# Office 2010 = -m 9500
# Office 2013+ = -m 9600
hashcat -m 9600 office.hash rockyou.txt
```

## KeePass

```bash
keepass2john database.kdbx > keepass.hash
john --wordlist=rockyou.txt keepass.hash
# hashcat -m 13400
```

## See also

- [[24 - John the Ripper Reference]]
- [[20 - Hashcat Mode IDs]]
