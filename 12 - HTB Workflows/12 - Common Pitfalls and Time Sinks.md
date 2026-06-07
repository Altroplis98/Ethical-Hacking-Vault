---
tags: [pentest, htb, antipattern, methodology, both]
type: workflow
---
# Common Pitfalls and Time Sinks

[[00 - README|Folder index]]

The mistakes that cost the most hours. If you catch yourself doing one of these, stop and reset.

## The 10 most expensive mistakes

### 1. Not adding hostnames to /etc/hosts

**Symptom:** Web app looks "broken," redirects to a hostname that doesn't resolve, vhosts return the same default page.

**Fix:**
```bash
# Add the hostname nmap returned + any redirects you see
echo "10.10.10.x  target.htb www.target.htb admin.target.htb" | sudo tee -a /etc/hosts
```

### 2. Running only the top-1000 nmap

**Symptom:** "I've enumerated everything and nothing's vulnerable" - and you didn't scan all 65535 ports.

**Fix:** Always run `-p-` once, even if slow. Custom services love port 1337, 8888, 31337, 50000.

### 3. Trusting nmap's first version detection

**Symptom:** nmap says "http" - it's actually a custom API. Or nmap says nothing - the port is actually a misidentified well-known service.

**Fix:**
```bash
# Aggressive version probe
nmap -sV --version-intensity 9 -p<port> ip
# And manual banner grab
nc -nv ip port
curl -kv https://ip:port/
```

### 4. Chasing an exploit before exhausting enumeration

**Symptom:** 3 hours pounding on Apache 2.4.49 path-traversal CVE while the box has FTP anon with a `creds.txt` file you skipped.

**Fix:** Finish enumerating every port before reading a single PoC. The "easy win" is usually somewhere quieter.

### 5. Skipping `sudo -l` after Linux foothold

**Symptom:** Two hours of LinPEAS analysis, kernel exploit hunting, etc. when `sudo -l` would have shown `NOPASSWD: /usr/bin/find`.

**Fix:** First command after `id` on any Linux shell: `sudo -l`.

### 6. Skipping `whoami /priv` after Windows foothold

**Symptom:** Hunting for service misconfigs while SeImpersonatePrivilege sits in your `whoami /priv` output.

**Fix:** First command on any Windows shell: `whoami /priv`. If you see SeImpersonatePrivilege, just go run Potato.

### 7. Brute-forcing instead of password-spraying

**Symptom:** Hydra locks out a target user; the account locks the engagement.

**Fix:**
```bash
# Check lockout first
nxc smb ip -u anyuser -p '' --pass-pol

# Spray, don't brute - one password against many users
nxc smb ip -u users.txt -p 'Welcome2026!' --continue-on-success
```

### 8. Re-running exploits without checking why they failed

**Symptom:** You ran the same `impacket-psexec` 8 times because "maybe it'll work this time."

**Fix:** Read the error.
- `STATUS_LOGON_FAILURE` → wrong cred
- `STATUS_ACCESS_DENIED` → cred is right, no rights on this service
- `STATUS_PASSWORD_EXPIRED` → password change needed (`smbpasswd`)
- `STATUS_ACCOUNT_DISABLED` → try a different user
- `KRB_AP_ERR_SKEW` → clock skew, sync time

### 9. Forgetting to re-enumerate after lateral movement

**Symptom:** You got `user1`, then `user2` via password reuse, then you ran the same `sudo -l` checks again - but as `user2` they're members of a different group with completely different access.

**Fix:** Treat every user as a new box:
```bash
id; sudo -l; ls /home; cat /etc/group | grep $(whoami)
groups
find / -group $(id -gn) -writable 2>/dev/null | head
```

### 10. Not making a real notes file

**Symptom:** "What command did I run to crack that hash an hour ago?"

**Fix:** Use `script` from session start, plus a markdown notes file from [[04 - Note-Taking Template]]. Both. Always.

## Smaller time-sinks

### Wasted time on wordlists

- `rockyou.txt` is the default, but `xato-net-10-million-usernames` for users, `top-shortlist-100`-style for first pass, `raft-medium-directories` for web.
- For default-cred attacks: `/usr/share/seclists/Passwords/Default-Credentials/`.
- Don't run rockyou as your *first* attempt. Try `top-100.txt` first - it ends fast and most easy boxes solve there.

### Wrong nmap scan type

- `-sS` (SYN, default with root) is fastest but requires root.
- `-sT` (connect) works without root and through Tor (proxychains).
- `-sU` is *slow* - top 100 is usually enough.

### `-Pn` confusion

- Add `-Pn` to skip ping. HTB blocks ICMP - `-Pn` is mandatory.
- On real engagements, `-Pn` makes the scan slower but more reliable.

### Wrong wordlist size for ffuf

- Start with `raft-small-directories.txt` (~20k entries) - finishes in a minute.
- Only move to `raft-medium` (~100k) or `raft-large` (~500k) if small returned nothing.

### Missing extensions

```bash
# Common extension permutations
gobuster dir -u ... -x php,html,txt,bak,old,zip,tar.gz,inc,cfg,conf,xml,json
# Add language-specific based on what whatweb told you
```

### IPv6 services missed

If you see weird "host not responding" patterns, scan IPv6 too:

```bash
sudo nmap -6 -p- -Pn <ipv6-addr>
```

### Burp not catching HTTPS

Install Burp's cert in Firefox/Chrome first. Otherwise HTTPS won't intercept.

## When you've been stuck >2 hours

1. Read your own notes back from the top.
2. Re-run nmap with `-A -p-`. New ports? New services?
3. Re-read every banner. Re-read every comment. Re-read every page source.
4. Ask: "If I were the box author, where would I have hidden it?"
5. Check the HTB forum (no spoilers) for general hint posts.
6. Sleep. Come back tomorrow.

> [!tip] The 1-hour rule
> If a single technique hasn't worked after 1 hour, switch techniques or go back to enumeration. You can always come back.
