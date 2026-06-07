---
tags: [pentest, scanning, nmap, evasion, firewall, both, recon]
tool: nmap
phase: 2
---
# nmap Firewall Evasion and Decoys

Techniques to get scan results past firewalls, IDS/IPS, and packet filters.

[[03 - Scanning/00 - README|Folder index]]

## Fragmentation

```bash
# Fragment packets (8-byte fragments)
sudo nmap -f 10.10.10.10

# Smaller fragments
sudo nmap -ff 10.10.10.10

# Custom MTU
sudo nmap --mtu 24 10.10.10.10
```

## Decoys

```bash
# Use decoy IPs (your scan hides among fake sources)
sudo nmap -D 10.10.10.1,10.10.10.2,ME,10.10.10.3 10.10.10.10

# Random decoys
sudo nmap -D RND:5 10.10.10.10
```

> [!warning] Decoys don't work with `-sT` (connect scan) or version detection
> Decoys only apply to raw packet scans like SYN scan.

## Source manipulation

```bash
# Spoof source port (use common ports that are often allowed)
sudo nmap --source-port 53 10.10.10.10     # DNS
sudo nmap --source-port 80 10.10.10.10     # HTTP
sudo nmap -g 53 10.10.10.10                # shorthand

# Spoof source IP (blind scan — you won't see responses)
sudo nmap -S 10.10.10.99 -e eth0 10.10.10.10

# MAC address spoofing
sudo nmap --spoof-mac 0 10.10.10.10        # random MAC
sudo nmap --spoof-mac Dell 10.10.10.10     # Dell vendor prefix
```

## Timing and stealth

```bash
# Slow scan (IDS evasion)
sudo nmap -T1 10.10.10.10     # Sneaky
sudo nmap -T0 10.10.10.10     # Paranoid (very slow)

# Custom timing
sudo nmap --scan-delay 5s 10.10.10.10
sudo nmap --max-rate 10 10.10.10.10
```

## Data length padding

```bash
# Append random data to packets (avoid signature matching)
sudo nmap --data-length 50 10.10.10.10
```

## Bypass techniques summary

| Technique | Flag | Defeats |
| --- | --- | --- |
| Fragmentation | `-f` | Simple packet inspection |
| Decoys | `-D` | Source-based blocking |
| Source port | `-g 53` | Port-based ACLs |
| Slow timing | `-T0`/`-T1` | Rate-based IDS |
| Data padding | `--data-length` | Signature-based IDS |
| Idle scan | `-sI` | All source-based detection |

## See also

- [[11 - nmap Timing Templates]] — timing in detail
- [[12 - nmap Idle (Zombie) Scan]] — ultimate source stealth
