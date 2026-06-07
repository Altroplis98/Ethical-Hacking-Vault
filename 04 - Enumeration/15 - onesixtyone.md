---
tags: [pentest, enumeration, snmp, onesixtyone, recon]
tool: onesixtyone
phase: 3
---
# onesixtyone

Fast SNMP community string brute-forcer. Discovers devices with default or weak SNMP community strings.

[[04 - Enumeration/00 - README|Folder index]]

## Install / verify

```bash
which onesixtyone || sudo apt install onesixtyone -y
```

## Usage

```bash
# Brute-force community strings on a single host
onesixtyone 10.10.10.10 -c /usr/share/seclists/Discovery/SNMP/common-snmp-community-strings.txt

# Sweep a range
onesixtyone -c /usr/share/seclists/Discovery/SNMP/common-snmp-community-strings.txt -i targets.txt
```

## Common community strings

```text
public
private
community
manager
cisco
admin
```

## Next step after finding a valid community string

```bash
# Full SNMP walk
snmpwalk -v 2c -c public 10.10.10.10
```

## See also

- [[16 - snmpwalk]] — deep SNMP enumeration after finding community string
- [[17 - snmp-check]] — friendlier SNMP output
