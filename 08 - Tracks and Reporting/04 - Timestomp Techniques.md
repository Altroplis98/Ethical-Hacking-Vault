---
tags: [pentest, tracks, anti-forensics, timestomp]
phase: 7
---

# Timestomp Techniques

Modifying file timestamps (MACE: Modified, Accessed, Created, Entry Modified) to make malicious files blend in.

[[08 - Tracks and Reporting/00 - README|Folder index]]

> [!danger] Study material — understand for detection engineering.

## Linux timestomping

```bash
# Change modification and access time to match another file
touch -r /bin/ls /tmp/malicious_file

# Set specific timestamp
touch -t 202301150830.00 /tmp/malicious_file

# Modify only access time
touch -a -t 202301150830.00 /tmp/malicious_file

# Modify only modification time
touch -m -t 202301150830.00 /tmp/malicious_file
```

## Windows timestomping

```powershell
# PowerShell — set creation time
$(Get-Item malware.exe).CreationTime = "01/15/2023 8:30:00 AM"
$(Get-Item malware.exe).LastWriteTime = "01/15/2023 8:30:00 AM"
$(Get-Item malware.exe).LastAccessTime = "01/15/2023 8:30:00 AM"
```

### Metasploit timestomp

```text
meterpreter > timestomp malware.exe -m "01/15/2023 08:30:00"
meterpreter > timestomp malware.exe -c "01/15/2023 08:30:00"
meterpreter > timestomp malware.exe -b  # blank all timestamps
```

## Detection

| Indicator | Method |
| --- | --- |
| $STANDARD_INFORMATION vs $FILE_NAME mismatch | MFT analysis (NTFS only) — $FILE_NAME timestamps are harder to modify |
| Creation time before compile time | PE header analysis |
| Timestamps before OS install date | Timeline analysis |
| Uniform timestamps across multiple files | Statistical analysis |

## See also

- [[01 - Linux Log Clearing Study]] — related anti-forensics
- [[05 - Detection Hooks (Purple Team)]] — detection strategies
