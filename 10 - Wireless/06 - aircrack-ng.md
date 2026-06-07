---
tags: [pentest, wireless, aircrack-ng, cracking, wpa, both, initial-access]
tool: aircrack-ng
phase: 5
---
# aircrack-ng

Crack WPA/WPA2 handshakes and WEP keys from captured packets. Dictionary-based for WPA; statistical for WEP.

[[10 - Wireless/00 - README|Folder index]]

## Install / verify

```bash
which aircrack-ng
sudo apt install aircrack-ng
```

## Crack WPA/WPA2 handshake (dictionary)

```bash
aircrack-ng -w /usr/share/wordlists/rockyou.txt capture-01.cap
```

| Flag | Meaning |
| --- | --- |
| `-w` | Wordlist path |
| `-b <BSSID>` | Target AP (if multiple in capture) |
| `-e <ESSID>` | Target by network name |
| `-l found.txt` | Write key to file |

## Pipe from other tools

```bash
# John the Ripper rules → aircrack-ng
john --wordlist=rockyou.txt --rules --stdout | aircrack-ng -w - capture-01.cap

# crunch generated passwords
crunch 8 8 0123456789 | aircrack-ng -w - capture-01.cap
```

> [!tip] Hashcat is faster
> aircrack-ng is CPU-only. For GPU cracking, convert the capture and use hashcat:
> ```bash
> # Convert .cap → .22000 for hashcat
> hcxpcapngtool capture-01.cap -o hash.22000
> hashcat -m 22000 hash.22000 rockyou.txt
> ```
> See [[08 - Hashcat WPA Mode 22000]].

## Crack WEP (statistical — no wordlist needed)

```bash
# Capture enough IVs first (50k+ data frames)
sudo airodump-ng -c <CH> --bssid <BSSID> -w wep wlan0mon

# Crack
aircrack-ng wep-01.cap
# or force PTW method:
aircrack-ng -z wep-01.cap
```

## Output

```text
KEY FOUND! [ 48:75:6E:74:65:72:32 ]   (ASCII: hunter2)
```

## See also

- [[04 - airodump-ng]]
- [[05 - aireplay-ng Deauth]]
- [[08 - Hashcat WPA Mode 22000]]
