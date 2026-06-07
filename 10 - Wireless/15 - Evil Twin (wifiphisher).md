---
tags: [pentest, wireless, evil-twin, phishing, wifiphisher, both, initial-access]
tool: wifiphisher
phase: 5
---
# Evil Twin — wifiphisher

Automated Evil Twin + captive portal phishing framework. Creates a rogue AP that mimics a target network and serves phishing pages to capture credentials or deploy payloads.

[[10 - Wireless/00 - README|Folder index]]

## Install / verify

```bash
which wifiphisher
sudo apt install wifiphisher
# or from source:
git clone https://github.com/wifiphisher/wifiphisher.git
cd wifiphisher && sudo python3 setup.py install
```

## How Evil Twin works

```text
1. Deauth clients from the real AP
2. Clone the target SSID on your adapter
3. Run a rogue DHCP + DNS server
4. Serve a captive portal page
5. Client connects to your fake AP (same name, stronger signal)
6. Client sees captive portal → enters credentials
7. You capture the credentials
```

## Basic usage (interactive)

```bash
sudo wifiphisher
# Interactive menu:
# 1. Select target AP
# 2. Choose phishing scenario
# 3. Attack runs automatically
```

## Command-line usage

```bash
sudo wifiphisher -aI wlan0 -eI wlan1 -p firmware-upgrade
```

| Flag | Meaning |
| --- | --- |
| `-aI` | Interface for rogue AP |
| `-eI` | Interface for deauth (needs injection) |
| `-p` | Phishing scenario |
| `-e <ESSID>` | Target ESSID to clone |
| `--handshake-capture` | Also capture WPA handshake |
| `-kN` | Don't deauth, just run rogue AP |

## Built-in phishing scenarios

| Scenario | Description |
| --- | --- |
| `firmware-upgrade` | "Router firmware update" — asks for WPA password |
| `oauth-login` | Fake OAuth login (Google/Facebook) |
| `plugin-update` | "Browser plugin required" — serves payload |
| `wifi-connect` | Generic WiFi connection portal |

> [!warning] Two adapters required
> One adapter runs the rogue AP, the other deauths the real AP. You need two adapters (or one that supports VIFs).

## Custom phishing pages

```bash
# Create custom scenario directory
mkdir -p /etc/wifiphisher/phishing-pages/my-scenario/
# Add: config.ini, index.html, static/
```

## See also

- [[16 - Fluxion]]
- [[17 - airgeddon]]
- [[05 - aireplay-ng Deauth]]
