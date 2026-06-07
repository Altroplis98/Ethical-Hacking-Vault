---
tags: [pentest, tracks, linux, logs, anti-forensics]
phase: 7
---

# Linux Log Clearing Study

Understanding how attackers clear Linux logs — so you can detect and document these techniques.

[[08 - Tracks and Reporting/00 - README|Folder index]]

> [!danger] Study material only
> In authorized engagements, you preserve logs. This exists so you understand detection opportunities.

## Key log locations

| Log file | Contents |
| --- | --- |
| `/var/log/auth.log` | Authentication events (SSH, sudo) |
| `/var/log/syslog` | General system messages |
| `/var/log/wtmp` | Login records (binary — use `last`) |
| `/var/log/btmp` | Failed login attempts (binary — use `lastb`) |
| `/var/log/lastlog` | Last login per user (binary — use `lastlog`) |
| `/var/log/apache2/` | Web server logs |
| `/var/log/nginx/` | Nginx logs |
| `~/.bash_history` | Command history per user |
| `/var/log/kern.log` | Kernel messages |
| `/var/log/cron.log` | Cron job execution |

## Clearing techniques (for study)

### History manipulation

```bash
# Clear current session history
history -c && history -w

# Unset history file
unset HISTFILE
export HISTFILESIZE=0

# Redirect history to /dev/null
ln -sf /dev/null ~/.bash_history

# Prepend space to hide individual commands (HISTCONTROL=ignorespace)
 sensitive_command_here
```

### Log file clearing

```bash
# Truncate (preserves file, clears content)
> /var/log/auth.log
cat /dev/null > /var/log/syslog
truncate -s 0 /var/log/auth.log

# Remove specific entries (more subtle)
sed -i '/10.10.14.5/d' /var/log/auth.log
grep -v "attacker_ip" /var/log/auth.log > /tmp/clean && mv /tmp/clean /var/log/auth.log
```

### Binary log clearing

```bash
# Clear wtmp/btmp (login records)
> /var/log/wtmp
> /var/log/btmp

# Use utmpdump to selectively edit
utmpdump /var/log/wtmp > /tmp/wtmp.txt
# Edit the text file, remove entries
utmpdump -r < /tmp/wtmp.txt > /var/log/wtmp
```

## Detection hooks

| Indicator | Detection method |
| --- | --- |
| Truncated log files (size drops to 0) | File integrity monitoring (AIDE, OSSEC) |
| Gaps in log timestamps | SIEM correlation, log gap alerting |
| Missing bash_history | User profile monitoring |
| `sed`/`grep` on log files | auditd rules on log directories |
| Log file permission changes | inotify watches |

## See also

- [[02 - Windows Event Log Clearing Study]] — Windows equivalent
- [[05 - Detection Hooks (Purple Team)]] — how to detect these techniques
