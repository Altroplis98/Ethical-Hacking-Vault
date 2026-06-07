---
tags: [pentest, htb, template, both]
type: template
aliases: [Box Template]
---
# Box Notes Template

Copy this file and rename to `<Boxname>.md` for each new box.

[[00 - README|Folder index]]

---

# {{boxname}}

**Difficulty:** Easy / Medium / Hard / Insane
**OS:** Linux / Windows
**Date started:** YYYY-MM-DD
**IP:** 10.10.10.x
**Hostname(s):** boxname.htb

## TL;DR (fill in last)

Foothold via: ___
User as ___ via: ___
Root via: ___

## Recon

### nmap

```bash
# Initial fast scan
sudo nmap -p- --min-rate 5000 -Pn 10.10.10.x -oA nmap/all

# Detailed on open ports
sudo nmap -sC -sV -p<ports> -Pn 10.10.10.x -oA nmap/full

# UDP top 100
sudo nmap -sU --top-ports 100 -Pn 10.10.10.x -oA nmap/udp
```

**Open ports:** 22, 80, ...

### /etc/hosts
```
10.10.10.x  boxname.htb www.boxname.htb
```

## Enumeration

### Port 80 - <Tech>

Findings:
- Server: ...
- Tech stack: ...
- Interesting endpoints: ...

```bash
# Commands run
ffuf -u http://boxname.htb/FUZZ -w ...
```

### Port 445 - SMB

Findings:
- Shares: ...
- Anon access: yes/no

### Port ... - ...

(repeat per service)

## Vulnerability identified

- ___ (with link to CVE / writeup if applicable)

## Foothold

### Path taken

1. ...
2. ...
3. shell as `<user>`

### Commands

```bash
# Exact reproduction
```

### Shell stabilization

```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
^Z
stty raw -echo; fg
export TERM=xterm
```

## User flag

```text
<user.txt contents - REDACTED in writeup>
```

## Lateral / user-to-user (if any)

1. Found cred in ___
2. Reused on ___
3. Now `<user2>`

## Priv-esc

### Enumeration output that mattered

- `sudo -l` →
- SUID find →
- Cron / process / writable →
- LinPEAS critical hit →

### Exploit

```bash
# Exact commands
```

## Root flag

```text
<root.txt contents - REDACTED in writeup>
```

## Loot / credentials harvested

| Source | User | Pass / Hash | Notes |
| --- | --- | --- | --- |
| /etc/shadow | root | $6$... | cracked: ... |
| db.sqlite | admin | hunter2 | reused on SSH |

## Screenshots checklist

- [ ] Initial nmap output
- [ ] First foothold shell
- [ ] User flag in shell with `whoami`
- [ ] Priv-esc proof
- [ ] Root flag in shell with `whoami`

## Lessons learned

- Tool I'll add to my toolkit: ___
- Trick I didn't know: ___
- What I'd do differently next time: ___
- Time spent stuck (and on what): ___

## References

- HTB writeup (after retire): ...
- Related CVE: ...
- Useful blog: ...
