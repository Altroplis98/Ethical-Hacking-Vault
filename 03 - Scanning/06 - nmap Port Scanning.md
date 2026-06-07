---
tags: [pentest, scanning, nmap, ports, tcp, udp, both, recon]
tool: nmap
phase: 2
---
# nmap Port Scanning

Deep dive into nmap scan types — how they work and when to use each.

[[03 - Scanning/00 - README|Folder index]]

## TCP scan types

### SYN scan (`-sS`) — default with root

```bash
sudo nmap -sS 10.10.10.10
```

Sends SYN, receives SYN/ACK (open) or RST (closed). Never completes the handshake — faster and stealthier than connect scan. Requires root.

### Connect scan (`-sT`) — default without root

```bash
nmap -sT 10.10.10.10
```

Full TCP three-way handshake. Slower, logged by target, but works without root.

### ACK scan (`-sA`) — firewall mapping

```bash
sudo nmap -sA 10.10.10.10
```

Doesn't determine open/closed — maps firewall rules. Responses of RST = unfiltered; no response = filtered.

### FIN / XMAS / NULL scans — IDS evasion

```bash
sudo nmap -sF 10.10.10.10    # FIN
sudo nmap -sX 10.10.10.10    # XMAS (FIN+PSH+URG)
sudo nmap -sN 10.10.10.10    # NULL (no flags)
```

Exploit RFC 793: closed ports respond with RST, open ports don't respond. Doesn't work against Windows (sends RST regardless).

### Window scan (`-sW`)

```bash
sudo nmap -sW 10.10.10.10
```

Like ACK scan but examines TCP window field in RST responses. Can sometimes differentiate open from closed even through firewalls.

## UDP scan

```bash
# UDP scan (slow — no handshake, relies on ICMP unreachable)
sudo nmap -sU 10.10.10.10

# Top UDP ports (faster)
sudo nmap -sU --top-ports 20 10.10.10.10

# Combined TCP + UDP
sudo nmap -sS -sU 10.10.10.10
```

> [!warning] UDP scanning is slow
> Open/filtered UDP ports don't respond, so nmap waits for timeout. Use `--top-ports 20` or target specific ports.

## Port states

| State | Meaning |
| --- | --- |
| open | Service accepting connections |
| closed | Reachable but no service listening |
| filtered | Firewall blocking probes |
| unfiltered | Reachable but can't determine open/closed (ACK scan) |
| open\|filtered | Can't determine (common in UDP) |

## See also

- [[05 - nmap Basics]] — fundamentals
- [[10 - nmap Firewall Evasion and Decoys]] — getting past filters
