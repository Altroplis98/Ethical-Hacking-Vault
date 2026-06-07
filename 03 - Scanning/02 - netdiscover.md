---
tags: [pentest, scanning, netdiscover, host-discovery, both, recon]
tool: netdiscover
phase: 2
---
# netdiscover

ARP-based host discovery with active and passive modes. Good for quietly mapping a local network.

[[03 - Scanning/00 - README|Folder index]]

## Install / verify

```bash
which netdiscover || sudo apt install netdiscover -y
```

## Usage

```bash
# Active scan — sends ARP requests
sudo netdiscover -r 10.10.10.0/24

# Passive mode — just listens for ARP traffic (stealthy)
sudo netdiscover -p

# Specify interface
sudo netdiscover -i eth0 -r 10.10.10.0/24

# Fast mode
sudo netdiscover -f -r 10.10.10.0/24
```

## Key flags

| Flag | Purpose |
| --- | --- |
| `-r range` | Target IP range (CIDR) |
| `-i iface` | Network interface |
| `-p` | Passive mode (listen only) |
| `-f` | Fast mode — don't wait between ARP requests |
| `-s time` | Sleep between requests (ms) |
| `-P` | Print results and exit (non-interactive) |

## Active vs. passive

| Mode | Stealth | Speed | Completeness |
| --- | --- | --- | --- |
| Active (`-r`) | Low — sends ARP requests | Fast | High |
| Passive (`-p`) | High — only listens | Slow (depends on traffic) | Low |

> [!tip] Passive mode on internal engagements
> Leave `netdiscover -p` running in the background while you work. It'll silently collect hosts as they communicate.

## See also

- [[01 - arp-scan]] — faster active ARP scanning
- [[04 - nmap Host Discovery]] — broader host discovery options
