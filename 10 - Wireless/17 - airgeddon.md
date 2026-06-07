---
tags: [pentest, wireless, airgeddon, evil-twin, all-in-one, both, initial-access]
tool: airgeddon
phase: 5
---
# airgeddon

All-in-one wireless security auditing framework with a text-based menu. Wraps many tools (aircrack-ng, mdk4, hashcat, bettercap, etc.) into a single guided workflow.

[[10 - Wireless/00 - README|Folder index]]

## Install / verify

```bash
git clone https://github.com/v1s1t0r1sh3r3/airgeddon.git
cd airgeddon
sudo bash airgeddon.sh
# Auto-checks and installs missing dependencies
```

## Feature overview

| Category | Capabilities |
| --- | --- |
| Recon | Interface selection, monitor mode, AP scanning |
| WPA/WPA2 | Handshake capture, dictionary, brute-force, PMKID |
| WPS | Reaver, Bully, Pixie Dust |
| Evil Twin | Captive portal, credential capture, BeEF integration |
| DoS | mdk4 deauth, auth flood |
| Decryption | Offline: aircrack-ng, hashcat, john |
| Enterprise | hostapd-wpe, credential harvesting |

## Typical workflow

```text
1. Launch airgeddon
2. Select wireless interface
3. Put in monitor mode (handled by menu)
4. Explore attack modules from the menu
5. Tool guides you through each step interactively
```

## Why use airgeddon?

- **Guided menus**: Don't need to remember every flag for every tool
- **Dependency management**: Auto-checks and installs what's needed
- **Integrated workflow**: Handshake capture → crack → Evil Twin in one session
- **Good for learning**: Shows you the underlying commands being run

> [!tip] Learning tool
> airgeddon is excellent for learning wireless attacks because it shows you exactly what commands it runs. Use it to learn, then run the tools directly for more control.

## See also

- [[15 - Evil Twin (wifiphisher)]]
- [[16 - Fluxion]]
- [[18 - EAPHammer Enterprise]]
