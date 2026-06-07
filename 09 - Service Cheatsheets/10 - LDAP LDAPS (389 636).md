---
tags: [pentest, cheatsheet, ldap, ad, service, active-directory, both]
port: [389, 636]
phase: reference
---
# LDAP / LDAPS (389 / 636)

[[09 - Service Cheatsheets/00 - README|Folder index]]

## Attacker Mindset

Active Directory is running. **Anonymous bind may be enabled** — if so, you can dump the entire directory unauthenticated. With any valid cred, enumerate users, groups, GPOs, ACLs, and trust relationships. This is where BloodHound data comes from. Pair with 3268 = Global Catalog = forest-wide queries. **Common attack vectors:** anonymous bind enumeration, LDAP injection, BloodHound collection, ACL abuse discovery, password spraying, LDAP relay.

## Enumerate

```bash
nmap -sV -sC -p 389,636 $IP
nmap --script ldap-rootdse -p 389 $IP
```

## Anonymous bind

```bash
ldapsearch -x -H ldap://$IP -s base namingcontexts
ldapsearch -x -H ldap://$IP -b "DC=corp,DC=local"
```

## Authenticated queries

```bash
ldapsearch -x -H ldap://$IP -D "user@corp.local" -w 'pass' -b "DC=corp,DC=local" "(objectClass=user)" sAMAccountName
```

## Tools

```bash
windapsearch -d corp.local -u user@corp.local -p 'pass' --dc-ip $IP -U   # Users
windapsearch -d corp.local -u user@corp.local -p 'pass' --dc-ip $IP -DA  # Domain Admins
nxc ldap $IP -u user -p 'pass' --users
```

> Full details: [[04 - Enumeration/07 - ldapsearch|ldapsearch]], [[04 - Enumeration/08 - windapsearch|windapsearch]]
