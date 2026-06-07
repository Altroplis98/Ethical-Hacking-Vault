---
tags: [pentest, wireless, moc, both]
type: reference
---
# 10 - Wireless

WiFi (WPA/WPA2/WPA3, WPS), deauth, Evil Twin, plus BLE/RFID basics.

[[00 - Vault Index|Home]]

> [!danger] Legal
> Deauth and jamming are illegal in many jurisdictions outside authorized testing. Test only on hardware you own or have explicit written authorization to test. 802.11w (PMF) defeats deauth.

## Files in this folder

### Setup
- [[01 - Monitor Mode Setup]]
- [[02 - WiFi Adapter Selection]]
- [[03 - airmon-ng]]

### WiFi attack tools
- [[04 - airodump-ng]]
- [[05 - aireplay-ng Deauth]]
- [[06 - aircrack-ng]]
- [[07 - hcxdumptool PMKID]]
- [[08 - Hashcat WPA Mode 22000]]
- [[09 - Reaver WPS]]
- [[10 - Bully WPS]]
- [[11 - Pixie Dust Attack]]
- [[12 - mdk4]]
- [[13 - Kismet]]
- [[14 - wash WPS Discovery]]

### Evil Twin / phishing
- [[15 - Evil Twin (wifiphisher)]]
- [[16 - Fluxion]]
- [[17 - airgeddon]]
- [[18 - EAPHammer Enterprise]]
- [[19 - hostapd-wpe]]

### Bluetooth / BLE
- [[20 - hcitool and bluetoothctl]]
- [[21 - gatttool BLE]]
- [[22 - btlejack]]
- [[23 - crackle BLE Pairing]]
- [[24 - Ubertooth One]]

### RFID / NFC
- [[25 - Proxmark3]]
- [[26 - Flipper Zero Overview]]
- [[27 - MIFARE Classic Attack]]
- [[28 - HID Prox 125kHz]]

## When you see WiFi in scope - flow

```text
1. Monitor mode + channel hop:  airmon-ng + airodump-ng
2. Identify target BSSID, channel, ENC type
3. WPA/WPA2:
     - Connected clients?  YES → deauth + capture 4-way → hashcat -m 22000
                            NO  → PMKID attack (hcxdumptool) → hashcat -m 22000
4. WPS-enabled (wash)?
     - Pixie Dust first (offline, fast)
     - Online Reaver brute as fallback
5. Enterprise (802.1X):
     - eaphammer karma + hostapd-wpe → capture MSCHAPv2 → asleap / hashcat -m 5500
6. Evil Twin only if scope allows captive-portal social engineering
```

> [!tip] Adapter recommendation
> Alfa AWUS036ACM (mt76x2u), AWUS036ACH (rtl8812au), or AWUS036NHA (ath9k) all support monitor mode + injection. The chipset matters more than the brand.
