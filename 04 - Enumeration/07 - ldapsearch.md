---
tags: [pentest, enumeration, ldap, ad, ldapsearch, active-directory, recon, windows]
tool: ldapsearch
phase: 3
---
# ldapsearch

Command-line LDAP client. Query Active Directory or any LDAP directory for users, groups, computers, GPOs, and more.

[[04 - Enumeration/00 - README|Folder index]]

## Basic queries

```bash
# Anonymous bind — enumerate base DN
ldapsearch -x -H ldap://10.10.10.10 -s base namingcontexts

# Anonymous enumeration (if allowed)
ldapsearch -x -H ldap://10.10.10.10 -b "DC=corp,DC=local"

# Authenticated
ldapsearch -x -H ldap://10.10.10.10 -D "user@corp.local" -w 'password' -b "DC=corp,DC=local"
```

## Common queries

### All users

```bash
ldapsearch -x -H ldap://10.10.10.10 -D "user@corp.local" -w 'pass' \
  -b "DC=corp,DC=local" "(objectClass=user)" sAMAccountName
```

### All computers

```bash
ldapsearch -x -H ldap://10.10.10.10 -D "user@corp.local" -w 'pass' \
  -b "DC=corp,DC=local" "(objectClass=computer)" dNSHostName operatingSystem
```

### Domain admins

```bash
ldapsearch -x -H ldap://10.10.10.10 -D "user@corp.local" -w 'pass' \
  -b "DC=corp,DC=local" "(&(objectClass=group)(cn=Domain Admins))" member
```

### Users with SPN (Kerberoastable)

```bash
ldapsearch -x -H ldap://10.10.10.10 -D "user@corp.local" -w 'pass' \
  -b "DC=corp,DC=local" "(&(objectClass=user)(servicePrincipalName=*))" sAMAccountName servicePrincipalName
```

### Users with no pre-auth (AS-REP Roastable)

```bash
ldapsearch -x -H ldap://10.10.10.10 -D "user@corp.local" -w 'pass' \
  -b "DC=corp,DC=local" "(&(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=4194304))" sAMAccountName
```

## Key flags

| Flag | Purpose |
| --- | --- |
| `-x` | Simple authentication |
| `-H ldap://host` | LDAP server URL |
| `-D bindDN` | Bind DN (user@domain or DN format) |
| `-w password` | Password |
| `-b baseDN` | Search base |
| `-s scope` | Search scope: base, one, sub |
| `"(filter)"` | LDAP filter |

## See also

- [[08 - windapsearch]] — friendlier LDAP enumeration
- [[09 - BloodHound]] — visual AD attack path mapping
