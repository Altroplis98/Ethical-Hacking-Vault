---
tags: [pentest, htb, checklist, both]
type: workflow
---
# Pre-Flight Checklist

Before you `nmap` anything, run this once per box session.

[[00 - README|Folder index]]

## Connectivity

```bash
# Are you on the VPN?
ip a | grep tun0
# tun0 should exist with HTB / OffSec / THM IP

# Can you reach the target?
ping -c 2 <target>          # may fail (-Pn later); not fatal
nc -zv <target> 80 443      # at least one common port should respond
```

## Workspace

```bash
# Per-box folder
mkdir -p ~/htb/<boxname>/{nmap,enum,exploit,loot,shells,screens}
cd ~/htb/<boxname>

# Tee everything (record what you typed and what it returned)
script -aqf ~/htb/<boxname>/session.log
# (Ctrl+D to end)
```

## Tooling sanity check (one-time, not per box)

```bash
which nmap nikto gobuster ffuf feroxbuster sqlmap hydra john hashcat
which crackmapexec nxc impacket-psexec evil-winrm xfreerdp responder
which wpscan nuclei searchsploit smbclient smbmap rpcclient
which subfinder amass assetfinder katana httpx dnsx

# Wordlists exist?
ls -la /usr/share/wordlists/rockyou.txt
ls -la /usr/share/seclists/ | head

# Searchsploit DB current?
searchsploit --update
nuclei -update-templates
```

## /etc/hosts

```bash
# After first nmap, if any hostname is returned:
echo "<target-ip>  <hostname.htb>  <hostname>" | sudo tee -a /etc/hosts
cat /etc/hosts | tail
```

## Listener ports ready

```bash
# Pick your default. Mine: 4444 / 9001 / 53 (egress-friendly)
sudo iptables -L INPUT -n | grep DROP        # firewall not blocking?
sudo ss -tlnp | grep -E ':(4444|9001|53)\s'  # nothing already listening?
```

## Mental check

- Scope confirmed in writing? (HTB ToS auto-covers HTB boxes; on real engagements verify written auth.)
- Time-box set? (Default: 2 hours total, then walk away.)
- Notebook open ([[04 - Note-Taking Template]])?
- "When you see X" cheat card open ([[01 - When You See X Do Y]])?

Then start.
