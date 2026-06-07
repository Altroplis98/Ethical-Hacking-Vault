---
tags: [pentest, wireless, wps, recon, wash, both]
tool: wash
phase: 5
---
# wash — WPS Discovery

Scan for APs with WPS enabled. Run this before any WPS attack to identify targets.

[[10 - Wireless/00 - README|Folder index]]

## Install / verify

```bash
which wash
# Part of reaver package
sudo apt install reaver
```

## Scan for WPS-enabled APs

```bash
sudo wash -i wlan0mon
```

Output columns:

| Column | Meaning |
| --- | --- |
| BSSID | AP MAC |
| Channel | Operating channel |
| RSSI | Signal strength |
| WPS Ver | WPS version (1.0 / 2.0) |
| WPS Locked | Whether WPS is locked after failed attempts |
| ESSID | Network name |

## Useful flags

| Flag | Meaning |
| --- | --- |
| `-i` | Monitor interface |
| `-5` | Scan 5 GHz channels too |
| `-s` | Scan for APs using Survey mode |
| `-a` | Show all APs (including those without WPS) |

## Interpreting results

| WPS Locked | Meaning | Action |
| --- | --- | --- |
| No | WPS accepting PINs | Attack with Reaver/Bully |
| Yes | WPS locked out | Wait, change MAC, or skip |

## Workflow

```text
1. wash -i wlan0mon          → find WPS-enabled APs
2. Pick target with WPS Locked = No
3. reaver -K (Pixie Dust)    → try offline attack first
4. If Pixie Dust fails       → reaver online brute force (slow)
5. If lockout occurs         → switch to Bully or wait
```

## See also

- [[09 - Reaver WPS]]
- [[10 - Bully WPS]]
- [[11 - Pixie Dust Attack]]
