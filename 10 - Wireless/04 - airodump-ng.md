---
tags: [pentest, wireless, aircrack-ng, recon, capture, both]
tool: airodump-ng
phase: 5
---
# airodump-ng

Wireless traffic sniffer and 802.11 frame capture tool. Shows all visible APs, clients, encryption types, and captures handshakes.

[[10 - Wireless/00 - README|Folder index]]

## Install / verify

```bash
which airodump-ng
# Part of aircrack-ng suite — preinstalled on Kali
```

## Basic scan (all channels, all bands)

```bash
sudo airodump-ng wlan0mon
```

Output columns:

| Column | Meaning |
| --- | --- |
| BSSID | AP MAC address |
| PWR | Signal strength (closer to 0 = stronger) |
| Beacons | Beacon frame count |
| #Data | Data frame count |
| #/s | Data frames per second |
| CH | Channel |
| MB | Max speed |
| ENC | Encryption (WPA2, WPA, WEP, OPN) |
| CIPHER | Cipher (CCMP, TKIP) |
| AUTH | Authentication (PSK, MGT, OPN) |
| ESSID | Network name |

## Target a specific AP and capture

```bash
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w capture wlan0mon
```

| Flag | Meaning |
| --- | --- |
| `-c 6` | Lock to channel 6 |
| `--bssid` | Filter to one AP |
| `-w capture` | Write to `capture-01.cap` |
| `--band abg` | Scan 2.4 + 5 GHz |
| `--wps` | Show WPS column |
| `--manufacturer` | Show device manufacturer |
| `--output-format pcap,csv` | Output formats |

## Capture a WPA handshake

```bash
# Terminal 1 — capture
sudo airodump-ng -c 6 --bssid <BSSID> -w handshake wlan0mon

# Terminal 2 — force a client to reconnect
sudo aireplay-ng -0 5 -a <BSSID> -c <CLIENT> wlan0mon

# Watch Terminal 1 for "WPA handshake: <BSSID>" in top-right
```

> [!tip] No clients visible?
> Use PMKID attack instead — see [[07 - hcxdumptool PMKID]]. It works without any connected clients.

## Output files

| Extension | Contents |
| --- | --- |
| `.cap` / `.pcap` | Raw packet capture (feed to aircrack-ng / hashcat) |
| `.csv` | AP and client summary |
| `.kismet.csv` | Kismet-compatible CSV |
| `.kismet.netxml` | Kismet XML format |

## See also

- [[05 - aireplay-ng Deauth]]
- [[06 - aircrack-ng]]
- [[07 - hcxdumptool PMKID]]
- [[13 - Kismet]]
