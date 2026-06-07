---
tags: [pentest, enumeration, snmp, snmp-check, recon]
tool: snmp-check
phase: 3
---
# snmp-check

Human-readable SNMP enumeration. Automatically categorizes output into system info, users, processes, shares, etc.

[[04 - Enumeration/00 - README|Folder index]]

## Install / verify

```bash
which snmp-check || sudo apt install snmp-check -y
```

## Usage

```bash
# Default (SNMPv1, community "public")
snmp-check 10.10.10.10

# SNMPv2c
snmp-check -v 2c 10.10.10.10

# Custom community string
snmp-check -c private 10.10.10.10

# Output to file
snmp-check 10.10.10.10 > snmp_results.txt
```

## Output categories

snmp-check organizes output into readable sections:
- System information (hostname, OS, uptime)
- User accounts
- Network information (interfaces, routes)
- Network services (listening ports)
- Processes
- Storage information
- Installed software
- Share information

> [!tip] snmp-check vs. snmpwalk
> snmp-check is better for quick human review. snmpwalk gives you raw OID data for deeper analysis or scripting.

## See also

- [[15 - onesixtyone]] — discover SNMP-enabled hosts
- [[16 - snmpwalk]] — raw SNMP walking
