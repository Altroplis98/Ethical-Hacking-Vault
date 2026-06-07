---
tags: [pentest, wireless, bluetooth, ubertooth, sniffing, both, initial-access]
tool: ubertooth
phase: 5
---
# Ubertooth One

Open-source 2.4 GHz radio platform for Bluetooth sniffing and research. The gold standard for Bluetooth security testing hardware.

[[10 - Wireless/00 - README|Folder index]]

## Install

```bash
sudo apt install ubertooth
ubertooth-util -v   # verify firmware version
```

## Update firmware

```bash
ubertooth-dfu -d bluetooth_rxtx.dfu -r
```

## Bluetooth Classic sniffing

```bash
# Sniff classic Bluetooth (LAP-level)
ubertooth-rx

# Follow a specific piconet
ubertooth-rx -l <LAP>
```

## BLE sniffing

```bash
# Sniff all BLE advertising channels
ubertooth-btle -f -c capture.pcap

# Follow a specific BLE connection
ubertooth-btle -f -t <BD_ADDR>

# Pipe to Wireshark in real-time
ubertooth-btle -f | wireshark -k -i -
```

| Flag | Meaning |
| --- | --- |
| `-f` | Follow connections |
| `-t <addr>` | Target specific device |
| `-c <file>` | Write PCAP |
| `-A` | Advertising channel only |
| `-p` | Promiscuous mode |

## Spectrum analysis

```bash
ubertooth-specan-ui
# Visual spectrum analyzer for 2.4 GHz band
# Shows WiFi, Bluetooth, Zigbee, microwaves, etc.
```

## Pentest use cases

- **BLE traffic capture**: Sniff unencrypted BLE data (IoT devices, smart locks)
- **Pairing capture**: Record pairing exchanges for [[23 - crackle BLE Pairing]]
- **Classic BT recon**: Identify Bluetooth devices and piconets
- **Spectrum analysis**: Find all 2.4 GHz emitters in an area

> [!tip] Worth the investment
> At ~$120, Ubertooth One is the most capable Bluetooth security tool available. If Bluetooth/BLE is in your pentest scope regularly, it pays for itself.

## See also

- [[20 - hcitool and bluetoothctl]]
- [[22 - btlejack]]
- [[23 - crackle BLE Pairing]]
