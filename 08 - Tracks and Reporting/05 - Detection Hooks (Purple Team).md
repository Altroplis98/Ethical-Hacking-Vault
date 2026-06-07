---
tags: [pentest, tracks, detection, purple-team, blue-team]
phase: 7
---

# Detection Hooks (Purple Team)

For each offensive technique in this vault, here are the corresponding detection opportunities. Bridge between red and blue.

[[08 - Tracks and Reporting/00 - README|Folder index]]

## Log clearing detection

| Attack | Detection |
| --- | --- |
| `wevtutil cl Security` | Event ID 1102 |
| Truncated Linux logs | AIDE/OSSEC file integrity alerts |
| Stopped Sysmon | Sysmon Event 255, service monitoring |
| Phant0m thread kill | Thread count monitoring, event gap detection |

## Credential attacks detection

| Attack | Detection |
| --- | --- |
| Kerberoasting | Event 4769 with RC4 encryption (0x17) |
| AS-REP Roasting | Event 4768 with RC4 encryption |
| Password spraying | Multiple 4771 events, same password hash |
| DCSync | Event 4662 with DS-Replication-Get-Changes |
| Mimikatz (sekurlsa) | LSASS access — Sysmon Event 10 |

## Lateral movement detection

| Attack | Detection |
| --- | --- |
| Pass-the-Hash | Event 4624 Type 3 with NTLM |
| PsExec | Event 7045 (service install) + named pipes |
| WMI execution | Event 4648, WMI activity |
| WinRM | Event 91, 168 in WinRM Operational |
| RDP | Event 4624 Type 10 |

## Persistence detection

| Attack | Detection |
| --- | --- |
| New scheduled task | Event 4698 |
| New service | Event 7045 |
| Registry run key | Sysmon Event 13 |
| Golden ticket | Event 4769 with non-existent account |
| DCShadow | Replication metadata anomalies |

## Network indicators

| Attack | Detection |
| --- | --- |
| Responder/LLMNR poisoning | Unexpected LLMNR/NBT-NS responses |
| DNS tunneling | High volume DNS TXT queries, entropy analysis |
| ICMP tunneling | ICMP packets with large payloads |
| Chisel/ligolo | Unusual outbound connections, beacon patterns |

## See also

- [[06 - Report Structure]] — document findings with detection recommendations
