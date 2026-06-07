---
tags: [pentest, pre-engagement, infrastructure, kali, setup, both]
phase: 0
---
# Attack Infrastructure Setup

Your toolbox needs to be sharp before you start. This is the "day before" checklist.

[[01 - Pre-Engagement/00 - README|Folder index]]

## Base OS

| Option | When to use |
| --- | --- |
| Kali Linux (VM or bare metal) | Default choice — everything pre-installed |
| Parrot OS | Lighter alternative, good for older hardware |
| Custom Ubuntu + tools | When you need a specific kernel or minimal footprint |
| Commando VM (Windows) | Windows-native tool testing (Rubeus, Seatbelt, etc.) |

## Day-before checklist

```bash
# 1. Update everything
sudo apt update && sudo apt full-upgrade -y

# 2. Verify key tools exist
for tool in nmap nikto gobuster ffuf hydra hashcat john sqlmap   msfconsole searchsploit responder impacket-secretsdump bloodhound   evil-winrm chisel ligolo-proxy enum4linux-ng; do
    which $tool 2>/dev/null || echo "MISSING: $tool"
done

# 3. Sync wordlists
ls -la /usr/share/wordlists/rockyou.txt
ls -la /usr/share/seclists/
# If SecLists missing:
sudo apt install seclists -y

# 4. Set up engagement directory
ENGDIR=~/engagements/$(date +%Y%m%d)_clientname
mkdir -p $ENGDIR/{scans,loot,screenshots,notes,exploits,evidence}
echo "Engagement started: $(date)" > $ENGDIR/notes/timeline.md
```

## VPN / network access

```bash
# Client VPN (OpenVPN)
sudo openvpn --config client.ovpn

# Verify you're on the right network
ip addr show tun0
ping -c 2 <first_in-scope_ip>

# HTB / TryHackMe
sudo openvpn --config lab.ovpn
```

> [!warning] Kill switch
> If your VPN drops, your real IP leaks to the target. Use a firewall rule:
> ```bash
> # Only allow traffic through tun0
> sudo iptables -A OUTPUT -o tun0 -j ACCEPT
> sudo iptables -A OUTPUT -o eth0 -d <vpn_server_ip> -j ACCEPT
> sudo iptables -A OUTPUT -o eth0 -j DROP
> ```

## Evidence directory structure

```text
engagement_root/
├── scans/          # nmap, masscan, nuclei output
├── loot/           # hashes, creds, tokens
├── screenshots/    # timestamped proof
├── notes/          # timeline, methodology notes
├── exploits/       # modified exploit code
└── evidence/       # packaged deliverables for report
```

## Tool-specific setup

```bash
# Metasploit DB
sudo msfdb init
msfconsole -q -x "db_status; exit"

# BloodHound (neo4j)
sudo neo4j start
# Default creds: neo4j / neo4j — change on first login

# Responder — edit config if needed
sudo nano /etc/responder/Responder.conf

# Burp Suite — configure browser proxy
# Firefox: Settings → Network → Manual Proxy → 127.0.0.1:8080
# Import Burp CA: http://burp → CA Certificate
```

## Anonymization (if required by ROE)

```bash
# Tor + proxychains
sudo apt install tor -y
sudo systemctl start tor
proxychains4 curl ifconfig.me

# Verify Tor is working
curl --socks5 127.0.0.1:9050 https://check.torproject.org/api/ip
```

## See also

- [[06 - Operational Logging]] — how to capture every command
- [[02 - Rules of Engagement]] — what you're allowed to do with this infrastructure
