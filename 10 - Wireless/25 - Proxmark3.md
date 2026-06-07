---
tags: [pentest, wireless, rfid, proxmark3, nfc, both, initial-access]
tool: proxmark3
phase: 5
---
# Proxmark3

The Swiss Army knife of RFID/NFC security research. Reads, clones, emulates, and attacks 125 kHz (LF) and 13.56 MHz (HF) cards.

[[10 - Wireless/00 - README|Folder index]]

## Install / verify

```bash
# Proxmark3 client
sudo apt install proxmark3
# or build from source for latest:
git clone https://github.com/RfidResearchGroup/proxmark3.git
cd proxmark3 && make clean && make all
```

## Connect

```bash
# Plug in Proxmark3 via USB
pm3     # launches the interactive client
# or specify port:
pm3 /dev/ttyACM0
```

## Low Frequency (125 kHz) — HID, EM4100, T55xx

```bash
# Auto-detect card type
[pm3] lf search

# Read HID Prox card
[pm3] lf hid read

# Clone HID card to T5577 blank
[pm3] lf hid clone --raw 2006ec0c86

# Read EM4100
[pm3] lf em 410x read

# Clone EM4100 to T5577
[pm3] lf em 410x clone --id 0102030405

# Brute-force HID facility codes
[pm3] lf hid brute --fc 42 --cn 1 --cn2 65535
```

## High Frequency (13.56 MHz) — MIFARE, NFC

```bash
# Auto-detect card type
[pm3] hf search

# Read MIFARE Classic
[pm3] hf mf autopwn    # auto key recovery + dump

# Dump all sectors
[pm3] hf mf dump

# Clone MIFARE Classic to magic card
[pm3] hf mf cload -f dump.bin

# Read NFC (NDEF)
[pm3] hf 14a read
```

## Emulation

```bash
# Emulate an HID card
[pm3] lf hid sim --raw 2006ec0c86

# Emulate MIFARE UID
[pm3] hf mf sim --uid 01020304
```

## Proxmark3 versions

| Model | LF | HF | Notes |
| --- | --- | --- | --- |
| PM3 Easy | Yes | Yes | Budget, limited range |
| PM3 RDV4 | Yes | Yes | Best range, built-in battery, BT |

> [!tip] Use RDV4 for physical pentests
> The RDV4 has a built-in battery and Bluetooth — you can clone a card while it's in your pocket, then walk to the reader and emulate it.

## See also

- [[26 - Flipper Zero Overview]]
- [[27 - MIFARE Classic Attack]]
- [[28 - HID Prox 125kHz]]
