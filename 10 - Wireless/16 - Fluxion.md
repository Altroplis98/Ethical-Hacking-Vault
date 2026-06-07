---
tags: [pentest, wireless, evil-twin, fluxion, wpa, both, initial-access]
tool: fluxion
phase: 5
---
# Fluxion

Automated Evil Twin WPA cracking framework. Captures a handshake, then creates a fake AP with a captive portal that asks for the WiFi password. Validates the entered password against the captured handshake in real-time.

[[10 - Wireless/00 - README|Folder index]]

## Install

```bash
git clone https://github.com/FluxionNetwork/fluxion.git
cd fluxion
sudo ./fluxion.sh
# First run installs dependencies automatically
```

## How Fluxion differs from wifiphisher

Fluxion validates the victim's input against the real WPA handshake. If the victim enters the wrong password, Fluxion shows an error and asks again. This means you get the real password, not just whatever the victim types.

## Workflow (interactive)

```bash
sudo ./fluxion.sh

# Menu flow:
# 1. Select language
# 2. Scan for targets (airodump)
# 3. Select target AP
# 4. Choose attack type (Captive Portal)
# 5. Capture handshake (auto-deauth + capture)
# 6. Select captive portal template
# 7. Attack launches — deauths real AP, runs fake AP
# 8. Victim connects to fake AP, enters WPA password
# 9. Fluxion checks password against handshake
# 10. If correct → saves and exits
```

## Requirements

- Two wireless adapters (one for rogue AP, one for deauth)
- Dependencies: aircrack-ng, hostapd, dnsmasq, iptables, lighttpd

> [!tip] Better than brute force
> If the password is long or complex (not in any wordlist), Fluxion can still recover it through social engineering — as long as someone enters it into the captive portal.

## See also

- [[15 - Evil Twin (wifiphisher)]]
- [[17 - airgeddon]]
- [[06 - aircrack-ng]]
