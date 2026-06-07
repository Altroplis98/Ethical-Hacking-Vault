---
tags: [pentest, cheatsheet, oracle, database, service, both]
port: 1521
phase: reference
---
# Oracle TNS (1521)

[[09 - Service Cheatsheets/00 - README|Folder index]]

## Attacker Mindset

Oracle listener. Check **default SIDs** (ORCL, XE, ORCLCDB). Default credentials are rampant historically (scott/tiger, system/manager, dbsnmp/dbsnmp, outln/outln). **TNS Poison** attacks redirect connections to attacker-controlled listeners. With SYSDBA, Java stored procedures give RCE. **Common attack vectors:** default credentials, SID enumeration (odat / tnscmd10g), TNS Poison, privilege escalation via SYSDBA, Java stored procedures for RCE.

## Enumerate

```bash
nmap -sV -sC -p 1521 $IP
nmap --script oracle-tns-version -p 1521 $IP
```

## ODAT (Oracle Database Attacking Tool)

```bash
pip install odat --break-system-packages

# SID enumeration
odat sidguesser -s $IP

# Credential brute-force
odat passwordguesser -s $IP -d SID

# All checks
odat all -s $IP -d SID -U user -P password
```

## Connect

```bash
sqlplus user/password@$IP:1521/SID
```

## Default credentials

```text
sys / change_on_install
system / manager
scott / tiger
dbsnmp / dbsnmp
```

## Key commands

```sql
SELECT * FROM v$version;
SELECT username FROM all_users;
SELECT * FROM user_tables;
```
