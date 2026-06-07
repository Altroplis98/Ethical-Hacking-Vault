---
tags: [pentest, scanning, nmap, host-discovery, both, recon]
tool: nmap
phase: 2
---
# nmap Host Discovery

Using nmap specifically for finding live hosts before doing port scans. Multiple probe types for different environments.

[[03 - Scanning/00 - README|Folder index]]

## Default discovery

```bash
# nmap default: ICMP echo + TCP SYN:443 + TCP ACK:80 + ICMP timestamp
nmap -sn 10.10.10.0/24
```

## Discovery probe types

```bash
# ICMP echo only
nmap -sn -PE 10.10.10.0/24

# TCP SYN ping on specific ports
nmap -sn -PS22,80,443 10.10.10.0/24

# TCP ACK ping (gets through stateless firewalls)
nmap -sn -PA80,443 10.10.10.0/24

# UDP ping (useful for DNS/SNMP hosts)
nmap -sn -PU53,161 10.10.10.0/24

# ARP ping (local subnet — most reliable)
nmap -sn -PR 10.10.10.0/24

# SCTP init ping
nmap -sn -PY 10.10.10.0/24

# Disable ping (treat all hosts as up — use when ICMP is blocked)
nmap -Pn 10.10.10.0/24
```

## Combining probes

```bash
# Best-effort discovery: ARP + ICMP + TCP SYN + TCP ACK
nmap -sn -PR -PE -PS22,80,443,445 -PA80,443 10.10.10.0/24

# Save alive hosts for later scanning
nmap -sn 10.10.10.0/24 -oG - | awk '/Up$/{print $2}' > alive.txt
```

## Key flags for discovery

| Flag | Probe type |
| --- | --- |
| `-sn` | No port scan (discovery only) |
| `-Pn` | Skip discovery (assume all up) |
| `-PE` | ICMP echo request |
| `-PP` | ICMP timestamp |
| `-PM` | ICMP address mask |
| `-PS<ports>` | TCP SYN |
| `-PA<ports>` | TCP ACK |
| `-PU<ports>` | UDP |
| `-PR` | ARP (local subnet) |
| `-PY<ports>` | SCTP INIT |
| `-n` | Don't resolve DNS (faster) |

## When to use `-Pn`

Use `-Pn` when:
- Target blocks all ICMP
- You know the host is up (e.g., you can see it in ARP)
- Cloud environments that filter discovery probes
- CTF / HTB where the box is guaranteed up

> [!warning] `-Pn` on a /24 is slow
> Without discovery, nmap port-scans every IP in the range. Only use on confirmed-alive hosts or small ranges.

## See also

- [[05 - nmap Basics]] — moving from discovery to port scanning
- [[01 - arp-scan]] — dedicated ARP discovery
- [[03 - fping]] — fast ICMP sweeps
