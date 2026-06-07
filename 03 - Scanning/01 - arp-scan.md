---
tags: [pentest, scanning, arp-scan, host-discovery, both, recon]
tool: arp-scan
phase: 2
---
# arp-scan

Layer 2 host discovery using ARP requests. Finds live hosts on the local subnet even if they block ICMP. Only works on the same broadcast domain.

[[03 - Scanning/00 - README|Folder index]]

## Install / verify

```bash
which arp-scan || sudo apt install arp-scan -y
```

## Usage

```bash
# Scan local subnet
sudo arp-scan -l

# Scan specific range
sudo arp-scan 10.10.10.0/24

# Specify interface
sudo arp-scan -I eth0 -l

# Resolve hostnames
sudo arp-scan -l -r 3

# Output to file
sudo arp-scan -l -x > arp_results.txt
```

## Key flags

| Flag | Purpose |
| --- | --- |
| `-l` | Scan local network (auto-detect range) |
| `-I iface` | Specify network interface |
| `-r N` | Retries per host |
| `-x` | Plain output (no header/footer) |
| `--localnet` | Same as `-l` |
| `-g` | Resolve IP to hostname |

## Why ARP scan?

- **Bypasses firewalls** — ARP operates at Layer 2; host-based firewalls typically don't block ARP
- **No false negatives** — if a host is on the LAN, ARP will find it
- **MAC address disclosure** — reveals NIC vendor (useful for identifying device types)

## Limitations

- Only works on the local subnet (not routable)
- Requires root/sudo
- Useless through a VPN tunnel (you're not on the same L2 segment)

## See also

- [[02 - netdiscover]] — similar ARP-based discovery with passive mode
- [[04 - nmap Host Discovery]] — works across subnets
