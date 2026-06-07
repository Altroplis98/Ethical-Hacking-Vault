---
tags: [pentest, wireless, hardware, adapter, both, recon]
type: reference
phase: 5
---
# WiFi Adapter Selection

Choosing the right USB adapter for wireless pentesting. The chipset matters more than the brand.

[[10 - Wireless/00 - README|Folder index]]

## Recommended chipsets

| Chipset | Band | Monitor | Injection | Driver | Notes |
| --- | --- | --- | --- | --- | --- |
| Atheros AR9271 | 2.4 GHz | Yes | Yes | `ath9k_htc` | Gold standard; works out of the box on Kali |
| MediaTek MT7612U | 2.4 + 5 GHz | Yes | Yes | `mt76x2u` | Best dual-band; kernel-native since 5.x |
| Realtek RTL8812AU | 2.4 + 5 GHz | Yes | Yes | `rtl8812au` (aircrack fork) | Needs manual driver install |
| Realtek RTL8814AU | 2.4 + 5 GHz | Yes | Yes | `rtl8814au` | Tri-stream; more power, bigger range |
| Ralink RT3070 | 2.4 GHz | Yes | Yes | `rt2800usb` | Legacy but reliable; cheap |

## Popular adapters by chipset

| Adapter | Chipset | Price tier |
| --- | --- | --- |
| Alfa AWUS036NHA | AR9271 | ~$25 |
| Alfa AWUS036ACM | MT7612U | ~$40 |
| Alfa AWUS036ACH | RTL8812AU | ~$50 |
| Panda PAU09 | RT5572 | ~$20 |
| TP-Link TL-WN722N **v1 only** | AR9271 | ~$15 |

> [!danger] TP-Link v2/v3 warning
> TP-Link TL-WN722N versions 2 and 3 use Realtek RTL8188EUS — no monitor mode, no injection. Only version 1 (AR9271) works. Check the FCC ID on the box before buying.

## Check your adapter

```bash
lsusb                            # find vendor:product ID
sudo airmon-ng                   # lists detected wireless interfaces
iw list | grep -A 10 "monitor"  # confirm monitor mode support
```

## Driver install (RTL8812AU example)

```bash
sudo apt install dkms
git clone https://github.com/aircrack-ng/rtl8812au.git
cd rtl8812au
sudo make dkms_install
sudo modprobe 88XXau
```

## See also

- [[01 - Monitor Mode Setup]]
- [[03 - airmon-ng]]
