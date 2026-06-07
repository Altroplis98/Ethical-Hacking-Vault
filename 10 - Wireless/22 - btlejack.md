---
tags: [pentest, wireless, ble, btlejack, sniffing, both, initial-access]
tool: btlejack
phase: 5
---
# btlejack

BLE sniffer and hijacker. Captures BLE traffic and can hijack active BLE connections. Requires a Micro:Bit (BBC) as the radio hardware.

[[10 - Wireless/00 - README|Folder index]]

## Install

```bash
pip install btlejack
# Flash the Micro:Bit firmware
btlejack -d      # detect Micro:Bit
btlejack -f      # flash firmware
```

## Hardware requirement

Needs 1-3 BBC Micro:Bit v1 or v2 boards (~$15 each). More boards = faster connection following across channels.

## Sniff BLE traffic

```bash
# Sniff a specific connection (need access address)
btlejack -c <access_address>

# Follow new connections
btlejack -f -o capture.pcap

# Sniff and save
btlejack -c <access_address> -o capture.pcap
```

## Hijack a BLE connection

```bash
# Jam and take over an existing BLE connection
btlejack -c <access_address> -t
```

> [!danger] Connection hijacking
> Hijacking disconnects the legitimate device. The target device owner will notice. Only use in authorized testing.

## Workflow

```text
1. Scan for BLE devices:     sudo hcitool lescan
2. Identify target connection
3. Sniff with btlejack:      btlejack -f -o capture.pcap
4. Analyze in Wireshark:     wireshark capture.pcap
5. Look for unencrypted data, credentials, commands
```

## See also

- [[21 - gatttool BLE]]
- [[23 - crackle BLE Pairing]]
- [[24 - Ubertooth One]]
