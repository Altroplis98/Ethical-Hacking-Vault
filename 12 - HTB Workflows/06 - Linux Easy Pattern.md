---
tags: [pentest, htb, linux, easy-box, walkthrough-pattern, both]
type: workflow
---
# Linux Easy Box - Walkthrough Pattern

Typical solution path for an HTB easy-rated Linux box. ~80% of easy Linux boxes follow one of these tracks.

[[00 - README|Folder index]]

## Track A: Web → reverse shell → sudo / SUID

Most common easy-box recipe.

```text
1. nmap reveals 22 + 80 (sometimes + 443)
2. Web app has obvious foothold:
     - CMS with known CVE (WordPress, Joomla, Drupal, OctoberCMS, etc.)
     - Default creds on /admin
     - File upload that accepts PHP
     - LFI to /proc/self/environ or log poisoning
     - Command injection in a "ping" or "lookup" feature
3. Pop a reverse shell as www-data / apache / nginx
4. PTY upgrade
5. Look in webroot for config files with creds → su to user
6. user.txt
7. sudo -l → GTFOBins → root.txt
```

### Track A drills

| Symptom | Likely path |
| --- | --- |
| WordPress version vulnerable | `wpscan` for known CVE; if you have any account → upload malicious plugin/theme |
| Joomla with `/administrator` | Default creds or known CVE → template editor RCE |
| Drupal 7 or 8 | Drupalgeddon (CVE-2018-7600) |
| OctoberCMS | CVE-2021-32648 auth bypass |
| WebUI port 8080/10000 (Webmin) | CVE-2019-15107 |
| `?file=`, `?include=`, `?page=` | LFI → log poisoning or `php://filter` source read → find creds in `.php` files |
| File upload form | Try double extension, magic byte, content-type swap; check disabled functions on shell |
| "Ping" / "DNS lookup" form | Command injection: `; id`, `\| id` |

## Track B: Anonymous service → cred → SSH

```text
1. nmap reveals 21 (FTP) + 22 + 80 or similar
2. Anonymous FTP allowed → download files
3. Files contain creds or hints (a note, a config, a backup)
4. SSH in as discovered user
5. user.txt
6. sudo -l or kernel exploit → root.txt
```

## Track C: Known service exploit

```text
1. nmap reveals a service with a known CVE on its exact version
   Common: ProFTPD 1.3.5 mod_copy, vsftpd 2.3.4, Samba <4.5.16,
           Apache Tomcat 7-9 (Ghostcat CVE-2020-1938),
           Drupal, Bolt CMS, Magento
2. searchsploit <product> <version>
3. Read the PoC, adjust LHOST/LPORT, run
4. Shell drops as some local user
5. Priv-esc as normal
```

## Linux easy priv-esc - what works ~90% of the time

Run these in order:

```bash
# 1. Sudo (the gimme)
sudo -l

# 2. SUID binaries cross-ref'd against GTFOBins
find / -perm -u=s -type f 2>/dev/null
# Quick filter for known winners
find / -perm -u=s -type f 2>/dev/null | grep -E "(vim|find|tar|nmap|less|more|awk|perl|python|node|env|cp|mv|chmod|chown|nano)"

# 3. Capabilities
getcap -r / 2>/dev/null

# 4. Writable files in cron
ls -la /etc/cron*; cat /etc/crontab
find /etc/cron* -writable 2>/dev/null

# 5. Process running as root that you can influence
pspy64

# 6. Group membership shortcuts
id
# Look for: docker, lxd, disk, video, root, sudo (if no PW), adm

# 7. Reused password
cat .bash_history; cat .ssh/known_hosts
# Try the user's web/db creds for su

# 8. If you have a Meterpreter shell (not just netcat/nc):
#    meterpreter > background
#    msf6 > use post/multi/recon/local_exploit_suggester
#    msf6 > set SESSION 1
#    msf6 > run
#    Catches kernel CVEs quickly. Also run linux-exploit-suggester.sh separately.
#    See [[15.5 - MSF Local Exploit Suggester]] and [[04 - Linux Exploit Suggester]]
```

> [!tip] The 5-minute checklist
> If you've got a Linux shell and haven't gotten root in 30 minutes, you almost certainly haven't run all 7 of the above. Run them in order, again, slowly. The answer is usually in step 1 (sudo -l) or step 2 (SUID).

## Easy-box red flags

These don't show up on easy boxes - if you're chasing one, you've gone down a rabbit hole:

- Kernel exploit chains
- Custom binary reverse engineering
- Pivoting / multiple hosts
- Cryptography puzzles deeper than a base64 → ROT13 → string
- Memory corruption / buffer overflow (rare on easy)
