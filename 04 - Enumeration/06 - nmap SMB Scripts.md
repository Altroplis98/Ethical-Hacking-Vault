---
tags: [pentest, enumeration, smb, nmap, nse, recon]
tool: nmap
phase: 3
---
# nmap SMB Scripts

NSE scripts specifically for SMB enumeration and vulnerability detection.

[[04 - Enumeration/00 - README|Folder index]]

## Enumeration scripts

```bash
# OS discovery via SMB
nmap --script smb-os-discovery -p 445 10.10.10.10

# Enumerate shares
nmap --script smb-enum-shares -p 445 10.10.10.10

# Enumerate users
nmap --script smb-enum-users -p 445 10.10.10.10

# Enumerate groups
nmap --script smb-enum-groups -p 445 10.10.10.10

# All SMB enum scripts
nmap --script "smb-enum-*" -p 445 10.10.10.10

# With credentials
nmap --script smb-enum-shares --script-args smbusername=user,smbpassword=pass -p 445 10.10.10.10
```

## Vulnerability scripts

```bash
# All SMB vuln checks
nmap --script "smb-vuln-*" -p 445 10.10.10.10

# Specific checks
nmap --script smb-vuln-ms17-010 -p 445 10.10.10.10   # EternalBlue
nmap --script smb-vuln-ms08-067 -p 445 10.10.10.10   # Conficker
nmap --script smb-vuln-cve-2017-7494 -p 445 10.10.10.10  # SambaCry
```

## SMB protocol info

```bash
# SMB protocols supported
nmap --script smb-protocols -p 445 10.10.10.10

# SMB security mode
nmap --script smb-security-mode -p 445 10.10.10.10

# SMB2 security mode
nmap --script smb2-security-mode -p 445 10.10.10.10
```

## See also

- [[05 - NetExec (nxc)]] — more comprehensive SMB tool
- [[01 - enum4linux and enum4linux-ng]] — wraps many of these checks
