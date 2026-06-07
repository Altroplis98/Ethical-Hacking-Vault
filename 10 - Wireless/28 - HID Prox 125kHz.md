---
tags: [pentest, wireless, rfid, hid, prox, cloning, both, initial-access]
tool: proxmark3
phase: 5
---
# HID Prox 125 kHz

HID ProxCard II and iCLASS are the most common physical access control cards in corporate environments. The 125 kHz Prox cards have zero cryptographic protection — they broadcast their ID in the clear.

[[10 - Wireless/00 - README|Folder index]]

## Why this matters

HID Prox 125 kHz cards:
- Transmit facility code + card number in plaintext
- No authentication, no encryption
- Can be read from several feet away (with the right antenna)
- Can be cloned to a T5577 blank card in seconds

## Read an HID Prox card

```bash
# Proxmark3
[pm3] lf hid read
# Output: HID Prox TAG ID: 2006ec0c86 (8342) - Format Len: 26bit - FC: 123 - Card: 45678

# Flipper Zero
# RFID → Read → hold card to back
```

## Clone to a T5577 blank

```bash
# Proxmark3
[pm3] lf hid clone --raw 2006ec0c86

# Or specify facility code + card number
[pm3] lf hid clone -w H10301 --fc 123 --cn 45678
```

T5577 cards cost ~$1 each and are rewritable.

## Long-range reading

With a purpose-built long-range antenna (e.g., custom LF coil), HID Prox cards can be read from 2-3 feet away. This enables:

- **Elevator/hallway capture**: Read badges as people walk by
- **Under-desk reader**: Hidden reader captures badges placed on desk
- **Weaponized Proxmark**: RDV4 with extended antenna in a bag

## Brute-force facility codes

```bash
# If you know a valid card number but not the facility code
[pm3] lf hid brute --fc 1 --fcmax 255 --cn 45678
# Tries all facility codes (0-255 for 26-bit format)
```

## HID card formats

| Format | Bits | FC range | CN range |
| --- | --- | --- | --- |
| H10301 (26-bit) | 26 | 0-255 | 0-65535 |
| H10302 (37-bit) | 37 | 0-65535 | 0-524287 |
| Corporate 1000 (35-bit) | 35 | 0-4095 | 0-1048575 |

## Mitigation recommendations

| Control | Effect |
| --- | --- |
| Upgrade to iCLASS SE / SEOS | Encrypted, mutual auth |
| Add PIN to card readers | Two-factor physical access |
| Enable tamper alerts | Detect rogue readers |
| Shielded badge holders | Block casual reading |

> [!danger] Critical finding
> If a client uses HID Prox 125 kHz for building access, this is a Critical finding in any physical security assessment. The cards can be cloned with $50 in equipment and zero skill.

## See also

- [[25 - Proxmark3]]
- [[26 - Flipper Zero Overview]]
- [[27 - MIFARE Classic Attack]]
