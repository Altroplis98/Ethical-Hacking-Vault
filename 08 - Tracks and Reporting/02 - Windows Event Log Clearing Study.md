---
tags: [pentest, tracks, windows, logs, anti-forensics]
phase: 7
---

# Windows Event Log Clearing Study

How attackers interact with Windows Event Logs — study material for detection engineering.

[[08 - Tracks and Reporting/00 - README|Folder index]]

> [!danger] Study material only — do NOT clear logs in authorized engagements.

## Key event logs

| Log | Path | Key event IDs |
| --- | --- | --- |
| Security | `Security.evtx` | 4624 (logon), 4625 (failed), 4672 (special privs), 4648 (explicit creds) |
| System | `System.evtx` | 7045 (service install), 7036 (service state) |
| PowerShell | `PowerShell/Operational` | 4103, 4104 (script block), 4688 (process create) |
| Sysmon | `Sysmon/Operational` | 1 (process), 3 (network), 11 (file create) |

## Clearing techniques (for study)

### wevtutil

```cmd
:: Clear specific log
wevtutil cl Security
wevtutil cl System
wevtutil cl Application

:: Clear all logs
for /F "tokens=*" %1 in ('wevtutil.exe el') DO wevtutil.exe cl "%1"
```

### PowerShell

```powershell
# Clear specific log
Clear-EventLog -LogName Security
Clear-EventLog -LogName System

# Clear all logs
Get-EventLog -LogName * | ForEach { Clear-EventLog $_.Log }
```

### Event ID 1102 — the clearing event

When the Security log is cleared, Windows generates **Event ID 1102** (Audit Log Cleared). This is the footprint that log clearing leaves behind.

```text
Event ID 1102: The audit log was cleared.
Subject:
  Security ID:  CORP\attacker
  Account Name: attacker
```

## Detection hooks

| Indicator | Event ID | Detection |
| --- | --- | --- |
| Security log cleared | 1102 | Alert on ANY 1102 event |
| System log cleared | 104 | Alert on 104 in System log |
| Log gap (missing time range) | — | SIEM time-series analysis |
| Sysmon stopped | 255 | Monitor Sysmon driver status |

## See also

- [[01 - Linux Log Clearing Study]] — Linux equivalent
- [[03 - Invoke-Phant0m]] — thread-killing technique
