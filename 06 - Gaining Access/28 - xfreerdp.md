---
tags: [pentest, rdp, xfreerdp, lateral, initial-access, windows]
tool: xfreerdp
phase: 5
---
# xfreerdp

The Linux RDP client preferred for pentesting. Handles PtH (with Restricted Admin), drive sharing, clipboard.

[[06 - Gaining Access/00 - README|Folder index]]

## Basic

```bash
xfreerdp /u:user /p:'pass' /v:<target>
xfreerdp /u:user /p:'pass' /v:<target> /dynamic-resolution
xfreerdp /u:user /p:'pass' /v:<target> /size:1920x1080
```

## Domain auth

```bash
xfreerdp /u:user /d:corp.local /p:'pass' /v:<target>
# or:
xfreerdp /u:'corp.local\user' /p:'pass' /v:<target>
```

## Pass-the-Hash (needs Restricted Admin enabled on target)

```bash
xfreerdp /u:admin /pth:<NTLM> /v:<target> /dynamic-resolution
# Enable Restricted Admin on target (need admin first):
# reg add "HKLM\System\CurrentControlSet\Control\Lsa" /v DisableRestrictedAdmin /t REG_DWORD /d 0
```

## Useful options

```bash
/drive:share,/tmp/share          # mount local /tmp/share as drive
/cert:ignore                     # ignore TLS errors
/sec:rdp                         # legacy RDP encryption (older targets)
/sec:nla                         # force NLA (default)
+clipboard                       # enable clipboard
/audio-mode:0                    # send audio
/log-level:debug                 # verbose
/multimon                        # multi-monitor
/sound:sys:pulse                 # forward sound
/microphone
```

## File transfer via clipboard

With `+clipboard`, copy text in your local terminal, paste into RDP. For binary files, use the `/drive:` mount or upload through another channel.

## When you see X, do Y

| Error | Fix |
| --- | --- |
| `Authentication failure, check credentials` | Wrong cred or NLA disabled. Try `/sec:rdp`. |
| `Could not connect to target` | Port 3389 closed; check `nmap -p3389`. |
| `PtH failed` | Target doesn't have Restricted Admin enabled. |
| TLS handshake failure | Add `/cert:ignore`. Or `/cert-tofu`. |
| Resolution looks bad | Use `/dynamic-resolution` instead of `/size:`. |

## Alternatives

- `rdesktop` - older; lacks PtH support
- `remmina` - GUI
- `Microsoft Remote Desktop` (Windows / macOS native)

> [!tip] Always use /dynamic-resolution
> Auto-fits the RDP window; works much better than fixed `/size:`.
