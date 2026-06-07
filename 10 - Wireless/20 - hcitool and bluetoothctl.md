---
tags: [pentest, wireless, bluetooth, hcitool, bluetoothctl, recon, both]
tool: hcitool
phase: 5
---
# hcitool and bluetoothctl

Built-in Linux Bluetooth utilities for scanning, pairing, and interacting with classic Bluetooth and BLE devices.

[[10 - Wireless/00 - README|Folder index]]

## Install / verify

```bash
which hcitool
which bluetoothctl
sudo apt install bluez
```

## hcitool — Classic Bluetooth scanning

```bash
# Scan for discoverable devices
hcitool scan

# BLE scan
sudo hcitool lescan

# Get device info
hcitool info <BD_ADDR>

# Get device name
hcitool name <BD_ADDR>
```

## bluetoothctl — Interactive Bluetooth control

```bash
sudo bluetoothctl

# Inside the interactive shell:
power on
scan on           # start scanning
scan off
devices           # list found devices
info <BD_ADDR>    # device details
pair <BD_ADDR>    # attempt pairing
connect <BD_ADDR> # connect
trust <BD_ADDR>   # auto-connect in future
remove <BD_ADDR>  # unpair
```

## Useful recon commands

```bash
# List local Bluetooth adapters
hciconfig -a

# Put adapter in discoverable mode
hciconfig hci0 piscan

# Read remote device features
hcitool info <BD_ADDR>
```

## Pentest use cases

- **Device enumeration**: Find Bluetooth devices in range (phones, IoT, headsets)
- **Service discovery**: `sdptool browse <BD_ADDR>` lists services
- **Pairing attacks**: Attempt pairing with default PINs (0000, 1234)
- **BLE recon**: Feed discovered devices to [[21 - gatttool BLE]] for deeper enumeration

> [!tip] hcitool is deprecated
> `hcitool` is being phased out in favor of `bluetoothctl` and the D-Bus API. Both still work on Kali, but `bluetoothctl` is more maintained.

## See also

- [[21 - gatttool BLE]]
- [[22 - btlejack]]
- [[24 - Ubertooth One]]
