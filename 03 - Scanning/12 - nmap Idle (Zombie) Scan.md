---
tags: [pentest, scanning, nmap, idle-scan, stealth, both, recon]
tool: nmap
phase: 2
---
# nmap Idle (Zombie) Scan

The ultimate stealth scan — uses a "zombie" host's IP ID sequence to scan a target without sending a single packet from your IP.

[[03 - Scanning/00 - README|Folder index]]

## How it works

1. Probe the zombie to learn its current IP ID value
2. Send a SYN to the target spoofed as the zombie
3. If the target port is open, it sends SYN/ACK to the zombie → zombie sends RST → zombie's IP ID increments
4. Probe the zombie again — if IP ID incremented by 2, the port was open; by 1, it was closed
5. Target never sees YOUR IP

## Usage

```bash
# Step 1: Find a good zombie (idle host with predictable IP ID)
nmap --script ipidseq 10.10.10.0/24

# Step 2: Run the idle scan
sudo nmap -sI <zombie_ip> 10.10.10.10
sudo nmap -sI <zombie_ip>:80 10.10.10.10  # use specific zombie port
```

## Finding good zombies

A good zombie must:
- Have **incremental** IP ID values (not random, not zero)
- Be **idle** (little traffic that would increment IP ID)
- Be **reachable** from both you and the target

```bash
# Check IP ID sequence
nmap --script ipidseq 10.10.10.50
# Look for: "IP ID Sequence Generation: Incremental"
```

## Limitations

- Slow (one round trip per port per probe)
- Requires a suitable zombie
- Only detects open vs. closed (no version/script detection)
- Some modern OSes use random IP IDs

## See also

- [[10 - nmap Firewall Evasion and Decoys]] — other evasion techniques
- [[06 - nmap Port Scanning]] — other scan types
