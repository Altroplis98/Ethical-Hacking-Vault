---
tags: [pentest, wireless, rfid, flipper-zero, multi-tool, both, initial-access]
tool: flipper-zero
phase: 5
---
# Flipper Zero Overview

Portable multi-protocol security tool with built-in support for sub-GHz, RFID (125 kHz + 13.56 MHz), NFC, infrared, iButton, GPIO, and BadUSB.

[[10 - Wireless/00 - README|Folder index]]

## Capabilities

| Protocol | Frequency | Read | Clone | Emulate |
| --- | --- | --- | --- | --- |
| RFID LF | 125 kHz | Yes | Yes (with writable cards) | Yes |
| NFC/HF | 13.56 MHz | Yes | Limited | Yes |
| Sub-GHz | 300-928 MHz | Yes | Yes | Yes |
| Infrared | IR | Yes | Yes | Yes |
| iButton | 1-Wire | Yes | Yes | Yes |
| BadUSB | USB HID | N/A | N/A | Yes |

## RFID / NFC usage

```text
# 125 kHz (LF)
RFID → Read → Hold card to back of Flipper
→ Saved → Emulate (hold Flipper to reader)

# 13.56 MHz (NFC)
NFC → Read → Hold card to Flipper
→ Detect Reader → learn what the reader asks for
→ Emulate
```

## Sub-GHz (garage doors, key fobs, weather stations)

```text
Sub-GHz → Read → Captures signal
→ Read RAW → captures raw signal (for rolling codes — record only)
→ Saved → Send (replay fixed codes)
```

> [!warning] Rolling codes
> Flipper can record rolling-code signals but replaying them only works once (the code is consumed). For older fixed-code systems (pre-2000 garage doors), replay works repeatedly.

## BadUSB (Rubber Ducky alternative)

```text
BadUSB → Select payload (.txt script)
→ Run → Flipper types the payload as a USB keyboard
```

Payload format is Ducky Script compatible.

## Flipper vs Proxmark3 for RFID

| Feature | Flipper Zero | Proxmark3 RDV4 |
| --- | --- | --- |
| Portability | Pocket-sized, battery | Larger, USB-powered |
| LF read range | Short | Much longer |
| HF attacks | Basic | Full (nested, hardnested, darkside) |
| MIFARE cracking | Limited | Full autopwn |
| Multi-protocol | Sub-GHz, IR, iButton, BadUSB | RFID only |
| Price | ~$170 | ~$300+ |

> [!tip] Complementary tools
> Flipper for quick field reads and multi-protocol work. Proxmark3 for serious RFID security research and complex attacks. Carry both.

## See also

- [[25 - Proxmark3]]
- [[27 - MIFARE Classic Attack]]
- [[28 - HID Prox 125kHz]]
