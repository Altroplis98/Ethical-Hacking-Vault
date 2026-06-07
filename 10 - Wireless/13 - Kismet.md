---
tags: [pentest, wireless, kismet, recon, ids, both]
tool: kismet
phase: 5
---
# Kismet

Wireless network detector, sniffer, wardriving tool, and WIDS (Wireless Intrusion Detection System). Supports WiFi, Bluetooth, Zigbee, and more via plugins.

[[10 - Wireless/00 - README|Folder index]]

## Install / verify

```bash
which kismet
sudo apt install kismet
```

## Start Kismet

```bash
# Web UI mode (default since Kismet 2019+)
sudo kismet -c wlan0mon

# Access web UI at http://localhost:2501
# Default creds: kismet / kismet (change immediately)
```

| Flag | Meaning |
| --- | --- |
| `-c wlan0mon` | Capture source |
| `-c wlan0mon:channel=6` | Lock to channel |
| `--no-logging` | Don't write log files |
| `-t 60` | Run for 60 seconds |
| `--override wardrive` | Use wardriving mode |

## Web UI features

- Live AP and client map
- Signal strength graphs
- Channel utilization
- Alert system (rogue APs, deauth detection)
- Export to various formats

## Kismet log files

| File | Format | Use |
| --- | --- | --- |
| `.kismet` | SQLite | Primary log — contains everything |
| `.pcapdump` | PCAP | Raw packets |
| `.wiglecsv` | CSV | Wardriving upload to WiGLE |

## Extract data from Kismet SQLite

```bash
kismetdb_to_pcap --in log.kismet --out packets.pcap
kismetdb_to_wiglecsv --in log.kismet --out wardriving.csv
```

## Use cases in pentests

- **Passive recon**: Identify all APs, clients, SSIDs, hidden networks without transmitting
- **Rogue AP detection**: Spot unauthorized APs in a corporate environment
- **Client probing**: See what SSIDs client devices are probing for (for Evil Twin)
- **Wardriving**: Map WiFi coverage for physical security assessments

> [!tip] Passive advantage
> Unlike airodump-ng, Kismet can operate purely passively — no injection, no deauth, no association. This makes it harder to detect during recon.

## See also

- [[04 - airodump-ng]]
- [[15 - Evil Twin (wifiphisher)]]
