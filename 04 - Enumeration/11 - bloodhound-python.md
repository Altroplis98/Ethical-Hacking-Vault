---
tags: [pentest, enumeration, ad, bloodhound-python, active-directory, recon, windows]
tool: bloodhound-python
phase: 3
---
# bloodhound-python

Python-based BloodHound collector. Runs from Linux — no domain join required. Just needs domain credentials.

[[04 - Enumeration/00 - README|Folder index]]

## Install

```bash
pip install bloodhound --break-system-packages
```

## Usage

```bash
# Full collection
bloodhound-python -u user -p 'password' -d corp.local -ns 10.10.10.10 -c all

# Specific collection
bloodhound-python -u user -p 'password' -d corp.local -ns 10.10.10.10 -c users,groups,computers

# With NTLM hash
bloodhound-python -u user --hashes aad3b435b51404ee:NTHASH -d corp.local -ns 10.10.10.10 -c all

# Zip output
bloodhound-python -u user -p 'password' -d corp.local -ns 10.10.10.10 -c all --zip
```

## Key flags

| Flag | Purpose |
| --- | --- |
| `-u user` | Username |
| `-p pass` | Password |
| `--hashes LM:NT` | NTLM hash |
| `-d domain` | Target domain |
| `-ns IP` | DNS server (usually DC) |
| `-c methods` | Collection methods (all, users, groups, etc.) |
| `--zip` | Compress output |
| `-dc dc_hostname` | Specify DC hostname |

## Output

Creates JSON files (or ZIP with `--zip`) compatible with BloodHound CE and legacy.

## See also

- [[09 - BloodHound]] — import and analyze the data
- [[10 - SharpHound]] — Windows alternative with more collection options
