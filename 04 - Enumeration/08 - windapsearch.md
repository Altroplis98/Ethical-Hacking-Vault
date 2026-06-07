---
tags: [pentest, enumeration, ldap, ad, windapsearch, active-directory, recon, windows]
tool: windapsearch
phase: 3
---
# windapsearch

Python-based LDAP enumeration tool with pre-built queries for common AD objects. Friendlier than raw ldapsearch.

[[04 - Enumeration/00 - README|Folder index]]

## Install

```bash
git clone https://github.com/ropnop/windapsearch.git
cd windapsearch
pip install -r requirements.txt --break-system-packages
# Or use the Go version:
# go install github.com/ropnop/go-windapsearch@latest
```

## Usage

```bash
# Domain users
python3 windapsearch.py -d corp.local -u user@corp.local -p 'pass' --dc-ip 10.10.10.10 -U

# Domain admins
python3 windapsearch.py -d corp.local -u user@corp.local -p 'pass' --dc-ip 10.10.10.10 -DA

# Domain computers
python3 windapsearch.py -d corp.local -u user@corp.local -p 'pass' --dc-ip 10.10.10.10 -C

# Groups
python3 windapsearch.py -d corp.local -u user@corp.local -p 'pass' --dc-ip 10.10.10.10 -G

# Privileged users
python3 windapsearch.py -d corp.local -u user@corp.local -p 'pass' --dc-ip 10.10.10.10 -PU

# Unconstrained delegation
python3 windapsearch.py -d corp.local -u user@corp.local -p 'pass' --dc-ip 10.10.10.10 --unconstrained-users
python3 windapsearch.py -d corp.local -u user@corp.local -p 'pass' --dc-ip 10.10.10.10 --unconstrained-computers

# Custom LDAP filter
python3 windapsearch.py -d corp.local -u user@corp.local -p 'pass' --dc-ip 10.10.10.10 --custom "(&(objectClass=user)(adminCount=1))"
```

## Key flags

| Flag | Purpose |
| --- | --- |
| `-U` | Enumerate users |
| `-G` | Enumerate groups |
| `-C` | Enumerate computers |
| `-DA` | Domain Admins |
| `-PU` | Privileged users |
| `--unconstrained-users` | Users with unconstrained delegation |
| `--custom "filter"` | Custom LDAP query |
| `-o file` | Output file |

## See also

- [[07 - ldapsearch]] — raw LDAP queries
- [[09 - BloodHound]] — graph-based AD analysis
