---
tags: [pentest, pre-engagement, logging, evidence]
phase: 0
---

# Operational Logging

Every command you run is potential evidence. If it's not logged, it didn't happen (or worse, you can't prove what DID happen).

[[01 - Pre-Engagement/00 - README|Folder index]]

## Why log everything

1. **Legal protection** — proves you stayed in scope
2. **Report accuracy** — exact commands, exact timestamps
3. **Reproducibility** — client asks "how did you get that hash?" and you can show them
4. **Timeline reconstruction** — correlate your actions with client's SIEM alerts

## Terminal logging with `script`

The simplest and most reliable method:

```bash
# Start logging (do this FIRST in every terminal)
LOGDIR=~/engagements/clientname/logs
mkdir -p $LOGDIR
script -t 2>$LOGDIR/timing_$(date +%s).log $LOGDIR/terminal_$(date +%s).log

# When done
exit  # ends the script session
```

### Replay a session

```bash
scriptreplay timing_file.log terminal_file.log
```

## tmux logging

If you use tmux (you should), enable automatic logging:

```bash
# In tmux, toggle logging for the current pane:
# Prefix + Shift-P (with tmux-logging plugin)

# Or manual pipe-pane:
tmux pipe-pane -o 'cat >> ~/engagements/clientname/logs/tmux_$(date +%Y%m%d_%H%M%S).log'
```

### tmux-logging plugin setup

```bash
# Install TPM (Tmux Plugin Manager)
git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm

# Add to ~/.tmux.conf:
set -g @plugin 'tmux-plugins/tmux-logging'
set -g @logging-path "$HOME/engagements/logs"

# Reload: Prefix + I
```

## Metasploit logging

```bash
# In msfconsole
spool /path/to/engagement/logs/msf_session.log

# Or start msfconsole with logging
msfconsole -L -o /path/to/msf.log
```

## Burp Suite logging

- Project → Project options → Misc → Logging
- Enable all logging categories
- Save project file frequently (`.burp` files contain full history)

## Screenshot discipline

```bash
# Linux screenshot with timestamp
import -window root ~/engagements/clientname/screenshots/$(date +%Y%m%d_%H%M%S).png

# Or use flameshot
flameshot full -p ~/engagements/clientname/screenshots/
```

### What to screenshot

| Event | Why |
| --- | --- |
| Initial access achieved | Proof of exploitation |
| Privilege escalation | Before/after showing user context |
| Sensitive data found | PoC (redact PII in report) |
| Domain admin achieved | The money shot |
| Every new foothold | Timeline evidence |

## Timestamp correlation

```bash
# Always note your timezone and NTP sync status
date
timedatectl status

# Use UTC for all logs if possible
TZ=UTC date
```

> [!tip] Ask the client for their SIEM timezone
> When they ask "what happened at 14:32?", you need to be in the same timezone.

## See also

- [[05 - Attack Infrastructure Setup]] — where your logs live
- [[07 - Communications Plan]] — when to communicate findings
