---
tags: [pentest, wireless, ble, gatttool, gatt, both]
tool: gatttool
phase: 5
---
# gatttool — BLE GATT Interaction

Command-line tool for interacting with Bluetooth Low Energy (BLE) devices via the GATT (Generic Attribute Profile) protocol. Read and write characteristics, enumerate services.

[[10 - Wireless/00 - README|Folder index]]

## Install / verify

```bash
which gatttool
sudo apt install bluez
```

## Connect to a BLE device

```bash
# Interactive mode
sudo gatttool -b <BD_ADDR> -I
[BD_ADDR][LE]> connect
[BD_ADDR][LE]> primary       # list services
[BD_ADDR][LE]> characteristics  # list characteristics
[BD_ADDR][LE]> char-read-hnd 0x000e   # read a handle
[BD_ADDR][LE]> char-write-req 0x000e 0100  # write to a handle
```

## Non-interactive (scripting)

```bash
# Read a characteristic
gatttool -b <BD_ADDR> --char-read -a 0x000e

# Write a value
gatttool -b <BD_ADDR> --char-write-req -a 0x000e -n 0100

# Listen for notifications
gatttool -b <BD_ADDR> --listen
```

## Common GATT handles

| Handle range | Typical service |
| --- | --- |
| 0x0001-0x000b | Generic Access (device name, appearance) |
| 0x000c-0x000f | Generic Attribute |
| 0x0010+ | Device-specific services |

## Pentest use cases

- **Smart locks**: Read/write lock state characteristics
- **IoT devices**: Enumerate services, find unprotected write handles
- **Fitness trackers**: Extract stored data
- **BLE beacons**: Read configuration

> [!warning] BLE security is often weak
> Many BLE devices have no authentication on GATT characteristics. If you can connect, you can often read and write everything. This is the finding you're looking for.

## See also

- [[20 - hcitool and bluetoothctl]]
- [[22 - btlejack]]
- [[23 - crackle BLE Pairing]]
