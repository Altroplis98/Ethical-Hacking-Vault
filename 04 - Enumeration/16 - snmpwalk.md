---
tags: [pentest, enumeration, snmp, snmpwalk, recon]
tool: snmpwalk
phase: 3
---
# snmpwalk

Walk an SNMP MIB tree to extract system info, network config, running processes, installed software, and sometimes credentials.

[[04 - Enumeration/00 - README|Folder index]]

## Install / verify

```bash
which snmpwalk || sudo apt install snmp -y
# Also install MIBs for readable OID names
sudo apt install snmp-mibs-downloader -y
sudo download-mibs
# Comment out "mibs :" in /etc/snmp/snmp.conf to enable MIB resolution
```

## Usage

```bash
# Full walk (verbose — lots of output)
snmpwalk -v 2c -c public 10.10.10.10

# Walk specific OID branches
snmpwalk -v 2c -c public 10.10.10.10 1.3.6.1.2.1.1        # System info
snmpwalk -v 2c -c public 10.10.10.10 1.3.6.1.4.1.77.1.2.25  # Windows users
snmpwalk -v 2c -c public 10.10.10.10 1.3.6.1.2.1.25.4.2.1.2  # Running processes
snmpwalk -v 2c -c public 10.10.10.10 1.3.6.1.2.1.6.13.1.3    # TCP open ports
snmpwalk -v 2c -c public 10.10.10.10 1.3.6.1.2.1.25.6.3.1.2  # Installed software
```

## High-value OIDs

| OID | Data |
| --- | --- |
| `1.3.6.1.2.1.1` | System description, uptime, contact |
| `1.3.6.1.4.1.77.1.2.25` | Windows user accounts |
| `1.3.6.1.2.1.25.4.2.1.2` | Running processes |
| `1.3.6.1.2.1.25.6.3.1.2` | Installed software |
| `1.3.6.1.2.1.6.13.1.3` | TCP listening ports |
| `1.3.6.1.2.1.2.2.1.2` | Network interfaces |
| `1.3.6.1.2.1.4.20.1.1` | IP addresses |

## SNMPv3

```bash
# SNMPv3 with auth and encryption
snmpwalk -v 3 -l authPriv -u username -a SHA -A 'authpass' -x AES -X 'privpass' 10.10.10.10
```

## See also

- [[15 - onesixtyone]] — find the community string first
- [[17 - snmp-check]] — friendlier output format
