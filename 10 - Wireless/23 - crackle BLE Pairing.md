---
tags: [pentest, wireless, ble, crackle, pairing, both, initial-access]
tool: crackle
phase: 5
---
# crackle — BLE Pairing Crack

Exploits weaknesses in BLE Legacy Pairing (BLE 4.0/4.1) to recover the Temporary Key (TK) and decrypt captured BLE traffic.

[[10 - Wireless/00 - README|Folder index]]

## Install

```bash
git clone https://github.com/mikeryan/crackle.git
cd crackle
make
sudo make install
```

## How it works

BLE Legacy Pairing uses a 6-digit TK (000000–999999) exchanged during pairing. crackle brute-forces all 1 million possibilities against a captured pairing exchange. This takes seconds.

> [!tip] Only works on Legacy Pairing
> BLE 4.2+ introduced LE Secure Connections (ECDH-based). crackle only works on the older Legacy Pairing. Check the pairing method in the capture.

## Usage

```bash
# 1. Capture BLE pairing exchange (Ubertooth or btlejack)
# 2. Run crackle against the capture
crackle -i pairing_capture.pcap -o decrypted.pcap
```

| Flag | Meaning |
| --- | --- |
| `-i` | Input PCAP with pairing exchange |
| `-o` | Output PCAP with decrypted traffic |
| `-l` | Specify the LTK directly (if known) |

## Output

```text
TK found: 000000
LTK found: 7f62c053a1b24e89...
Decrypting with LTK...
Wrote decrypted packets to decrypted.pcap
```

## See also

- [[21 - gatttool BLE]]
- [[22 - btlejack]]
- [[24 - Ubertooth One]]
