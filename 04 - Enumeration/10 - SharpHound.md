---
tags: [pentest, enumeration, ad, sharphound, bloodhound, active-directory, recon, windows]
tool: sharphound
phase: 3
---
# SharpHound

The official BloodHound data collector for Windows. Runs on a domain-joined machine to gather AD relationships.

[[04 - Enumeration/00 - README|Folder index]]

## Download

SharpHound is distributed with BloodHound releases. Get it from the BloodHound GitHub releases page.

## Usage

```powershell
# Default collection (Group, LocalAdmin, Session, Trusts)
.\SharpHound.exe -c All

# Specific collection methods
.\SharpHound.exe -c Group,LocalAdmin,Session

# Stealth collection (slower, less noise)
.\SharpHound.exe -c All --stealth

# Target specific domain
.\SharpHound.exe -c All -d corp.local

# Loop session collection (run for hours to catch more sessions)
.\SharpHound.exe -c Session --loop --loopduration 02:00:00
```

## Collection methods

| Method | What it collects |
| --- | --- |
| `Group` | Group memberships |
| `LocalAdmin` | Local admin relationships |
| `Session` | Active sessions (who's logged in where) |
| `Trusts` | Domain trust relationships |
| `ACL` | Object ACL/ACE permissions |
| `ObjectProps` | User/computer properties |
| `SPNTargets` | Kerberoastable targets |
| `Container` | OU/container structure |
| `GPOLocalGroup` | GPO-defined local group membership |
| `All` | Everything above |

## Output

SharpHound creates a ZIP file containing JSON files. Import this directly into BloodHound.

## OPSEC considerations

- Session collection queries every computer — generates significant traffic
- `--stealth` mode is slower but queries fewer hosts
- Consider running during business hours when session data is richest

## See also

- [[09 - BloodHound]] — where you import the data
- [[11 - bloodhound-python]] — collect from Linux without domain join
