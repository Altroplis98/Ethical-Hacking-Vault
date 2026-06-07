---
tags: [pentest, recon, osint, metagoofil, metadata, both]
tool: metagoofil
phase: 1
---
# Metagoofil

Downloads public documents (PDF, DOC, XLS, PPT) from a target domain and extracts metadata — usernames, software versions, file paths.

[[02 - Reconnaissance and OSINT/00 - README|Folder index]]

## Install / verify

```bash
which metagoofil || sudo apt install metagoofil -y
```

## Usage

```bash
# Download and extract metadata from PDFs and DOCs
metagoofil -d example.com -t pdf,doc,docx,xls,xlsx,ppt,pptx -l 100 -n 50 -o /tmp/meta -f results.html

# Key flags
# -d domain        Target domain
# -t filetypes     Comma-separated file types
# -l N             Search limit (max results per filetype)
# -n N             Max files to download
# -o dir           Output directory for downloaded files
# -f file          Output report file
```

## What metadata reveals

| Metadata field | Intelligence value |
| --- | --- |
| Author / Creator | Real employee names → username generation |
| Software version | "Microsoft Office 16.0" → OS/software inventory |
| File paths | `C:\Users\jsmith\Documents\` → username + OS |
| Email addresses | Embedded in document properties |
| Print server names | Internal infrastructure naming conventions |
| GPS coordinates | In images embedded in documents |

## Manual extraction with exiftool

```bash
# After downloading files with metagoofil, extract deeper metadata
exiftool -r /tmp/meta/ | grep -iE '(author|creator|producer|company|email|path|gps)'
```

## See also

- [[24 - ExifTool]] — deeper manual metadata extraction
- [[14 - Google Dorking]] — find documents via `filetype:` dorks
