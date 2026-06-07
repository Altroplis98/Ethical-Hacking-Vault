---
tags: [pentest, wireless, aircrack-ng, monitor-mode, both, recon]
tool: airmon-ng
phase: 5
---
# airmon-ng

Part of the aircrack-ng suite. Automates putting an adapter into monitor mode and killing interfering processes.

[[10 - Wireless/00 - README|Folder index]]

## Install / verify

```bash
which airmon-ng
airmon-ng --help
# Kali ships aircrack-ng suite by default
sudo apt install aircrack-ng   # if missing
```

## Core workflow

```bash
# 1. List wireless interfaces
sudo airmon-ng

# 2. Kill processes that interfere with monitor mode
sudo airmon-ng check kill

# 3. Start monitor mode
sudo airmon-ng start wlan0
# creates wlan0mon (or wlan0 stays, depending on driver)

# 4. Verify
iwconfig wlan0mon   # Mode:Monitor

# 5. When done — stop monitor mode
sudo airmon-ng stop wlan0mon

# 6. Restart networking
sudo systemctl start NetworkManager
```

## Start on a specific channel

```bash
sudo airmon-ng start wlan0 6     # lock to channel 6
sudo airmon-ng start wlan0 36    # lock to 5 GHz channel 36
```

## Common flags

| Flag / usage | Meaning |
| --- | --- |
| `airmon-ng` | List interfaces |
| `airmon-ng check` | List interfering processes |
| `airmon-ng check kill` | Kill interfering processes |
| `airmon-ng start <iface>` | Enable monitor mode |
| `airmon-ng start <iface> <ch>` | Monitor mode on a specific channel |
| `airmon-ng stop <iface>mon` | Disable monitor mode |

## Troubleshooting

| Symptom | Fix |
| --- | --- |
| Interface not listed | Adapter not recognized — check `lsusb` and drivers |
| `check kill` doesn't help | Manually `systemctl stop NetworkManager wpa_supplicant` |
| Monitor interface has wrong name | Some drivers keep `wlan0` — check `iwconfig` |

## See also

- [[01 - Monitor Mode Setup]]
- [[04 - airodump-ng]]
- [[05 - aireplay-ng Deauth]]
