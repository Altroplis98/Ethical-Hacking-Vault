---
tags: [pentest, tracks, windows, anti-forensics, phant0m]
phase: 7
---

# Invoke-Phant0m

Kills Windows Event Log service threads instead of stopping the service — logs stop being written but the service appears running.

[[08 - Tracks and Reporting/00 - README|Folder index]]

> [!danger] Study material only — understand this for detection, not operational use.

## How it works

1. Identifies the Event Log service process (svchost.exe hosting EventLog)
2. Enumerates threads within that process
3. Kills the threads responsible for writing events
4. Service status shows "Running" but no events are recorded

## Detection

| Indicator | How to detect |
| --- | --- |
| Event Log threads terminated | Monitor thread count of EventLog svchost |
| No new events despite activity | SIEM gap detection — if logon events stop, alert |
| Process manipulation | Sysmon Event ID 8 (CreateRemoteThread) |
| Service health check | Custom health check that writes a test event and verifies it appears |

## Defensive countermeasure

Forward logs in real-time to a SIEM or syslog collector. Even if local logging is killed, forwarded events are already shipped.

## See also

- [[02 - Windows Event Log Clearing Study]] — traditional log clearing
- [[05 - Detection Hooks (Purple Team)]] — detection strategies
