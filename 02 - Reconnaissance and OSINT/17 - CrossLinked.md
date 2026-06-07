---
tags: [pentest, recon, osint, linkedin, crosslinked, both]
tool: crosslinked
phase: 1
---
# CrossLinked

LinkedIn employee enumeration via search engine scraping. Builds username lists without needing a LinkedIn account.

[[02 - Reconnaissance and OSINT/00 - README|Folder index]]

## Install

```bash
pip install crosslinked --break-system-packages
```

## Usage

```bash
# Basic enumeration — outputs usernames in first.last format
crosslinked -f '{first}.{last}' "Example Company" -o employees.txt

# Different username formats
crosslinked -f '{f}{last}' "Example Company"      # jsmith
crosslinked -f '{first}.{last}' "Example Company"  # john.smith
crosslinked -f '{first}{l}' "Example Company"      # johns

# Use Bing instead of Google (or both)
crosslinked -f '{first}.{last}' "Example Company" --engine bing
```

## Format tokens

| Token | Output |
| --- | --- |
| `{first}` | Full first name |
| `{last}` | Full last name |
| `{f}` | First initial |
| `{l}` | Last initial |

## Why this matters

- Generates valid AD username lists for password spraying
- No LinkedIn account needed (scrapes via Google/Bing)
- Pair with [[06 - Gaining Access/05 - Password Spraying Strategy|password spraying]] for initial access

```bash
# Generate usernames → spray
crosslinked -f '{first}.{last}' "Target Corp" -o users.txt
nxc smb 10.10.10.5 -u users.txt -p 'Spring2024!' --continue-on-success
```

## See also

- [[18 - holehe]] — check which services an email is registered on
- [[19 - h8mail]] — find breached credentials
