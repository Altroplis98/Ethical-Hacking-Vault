---
tags: [pentest, wireless, monitor-mode, setup, both, recon]
tool: iw
phase: 5
---
# Monitor Mode Setup

Switch a WiFi adapter from managed (client) mode to monitor (promiscuous) mode so it can capture all 802.11 frames on a channel.

[[10 - Wireless/00 - README|Folder index]]

## Prerequisites

- A wireless adapter whose chipset supports monitor mode + packet injection (ath9k, mt76x2u, rtl8812au)
- Root / sudo
- No network-manager fighting over the interface

## Quick method — airmon-ng

```bash
sudo airmon-ng check kill      # kill interfering processes
sudo airmon-ng start wlan0     # creates wlan0mon
iwconfig wlan0mon              # confirm Mode:Monitor
```

When done:

```bash
sudo airmon-ng stop wlan0mon
sudo systemctl start NetworkManager
```

## Manual method — iw / ip

```bash
sudo ip link set wlan0 down
sudo iw dev wlan0 set type monitor
sudo ip link set wlan0 up
iw dev wlan0 info              # type should say "monitor"
```

Revert:

```bash
sudo ip link set wlan0 down
sudo iw dev wlan0 set type managed
sudo ip link set wlan0 up
sudo systemctl start NetworkManager
```

## Lock to a specific channel

```bash
sudo iw dev wlan0mon set channel 6
# or with airmon-ng:
sudo airmon-ng start wlan0 6
```

> [!warning] Interfering processes
> `wpa_supplicant`, `NetworkManager`, and `avahi-daemon` will pull the adapter back to managed mode. Always run `airmon-ng check kill` first, or manually stop them.

## Verify injection works

```bash
sudo aireplay-ng --test wlan0mon
# "Injection is working!" = good
```

## Troubleshooting

| Symptom | Fix |
| --- | --- |
| `iw` says "command failed: Device or resource busy" | Kill interfering processes first |
| Adapter disappears after `set type monitor` | Driver issue — try `modprobe -r <driver> && modprobe <driver>` |
| `aireplay-ng --test` fails injection | Chipset may not support injection; try a different adapter |
| `wlan0mon` not created | Older aircrack — use manual `iw` method |

## See also

- [[02 - WiFi Adapter Selection]]
- [[03 - airmon-ng]]
- [[04 - airodump-ng]]
