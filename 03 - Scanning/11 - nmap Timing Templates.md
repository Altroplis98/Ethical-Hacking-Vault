---
tags: [pentest, scanning, nmap, timing, performance, both, recon]
tool: nmap
phase: 2
---
# nmap Timing Templates

Control scan speed vs. stealth vs. accuracy with `-T` templates or granular timing options.

[[03 - Scanning/00 - README|Folder index]]

## Templates

| Template | Name | Use case |
| --- | --- | --- |
| `-T0` | Paranoid | IDS evasion (one probe every 5 min) |
| `-T1` | Sneaky | IDS evasion (15 sec between probes) |
| `-T2` | Polite | Reduce network load |
| `-T3` | Normal | Default — good balance |
| `-T4` | Aggressive | Fast on reliable networks |
| `-T5` | Insane | Fastest, may miss ports |

## Practical recommendations

```bash
# HTB / Lab (reliable network, speed matters)
nmap -T4 -p- 10.10.10.10

# Real engagement (balance speed and accuracy)
nmap -T3 10.10.10.10

# Stealth engagement
nmap -T1 10.10.10.10
```

## Granular timing options

```bash
--min-rate N          # Minimum packets per second
--max-rate N          # Maximum packets per second
--min-parallelism N   # Minimum parallel probes
--max-parallelism N   # Maximum parallel probes
--host-timeout Ns     # Give up on a host after N seconds
--scan-delay Ns       # Minimum delay between probes
--max-retries N       # Max retransmissions
```

### Recommended fast scan

```bash
sudo nmap -p- --min-rate 5000 -Pn 10.10.10.10
```

This sends at least 5000 packets/second — scans all 65535 ports in under 30 seconds on a good connection.

## See also

- [[10 - nmap Firewall Evasion and Decoys]] — evasion techniques
- [[05 - nmap Basics]] — core scan commands
