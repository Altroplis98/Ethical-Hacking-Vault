---
tags: [pentest, shells, pty, upgrade, tty, initial-access]
type: cheatsheet
phase: 5
---
# PTY Upgrade Ritual

Turn a dumb reverse shell into a fully interactive TTY with tab completion, arrow keys, Ctrl+C safety, and job control.

[[00 - README|Folder index]]

## The standard upgrade (memorize this)

```bash
# Step 1 — spawn a PTY inside the shell
python3 -c 'import pty; pty.spawn("/bin/bash")'
# alternatives if python3 is missing:
python -c 'import pty; pty.spawn("/bin/bash")'
script -qc /bin/bash /dev/null
/usr/bin/script -qc /bin/bash /dev/null

# Step 2 — background the shell
# Press Ctrl+Z

# Step 3 — fix your local terminal
stty raw -echo; fg

# Step 4 — set terminal type and size (inside the shell)
export TERM=xterm-256color
stty rows 40 cols 160
```

> [!tip] Get your terminal size
> On your attacker machine, run `stty size` to get the exact rows and columns. Use those values in step 4.

## Why each step matters

| Step | What it does |
| --- | --- |
| `pty.spawn` | Creates a pseudo-terminal — enables password prompts, su, ssh, etc. |
| `Ctrl+Z` | Suspends the remote shell so you can fix your local terminal |
| `stty raw -echo` | Passes raw keystrokes to the remote (arrow keys, tab, Ctrl+C) |
| `fg` | Brings the remote shell back to foreground |
| `export TERM` | Enables clear, colors, and ncurses apps |
| `stty rows/cols` | Fixes line wrapping |

## If things break

```bash
# Shell garbled after upgrade attempt? Reset:
reset
# or:
stty sane
```

## Alternative: socat full TTY

If socat is on the target (or you can upload it), skip the ritual entirely — see [[06 - socat Full TTY]].

## Alternative: pwncat-cs

Automated upgrade on connection — see [[05 - pwncat-cs Listener]].

## See also

- [[01 - Linux Reverse Shells]]
- [[05 - pwncat-cs Listener]]
- [[06 - socat Full TTY]]
