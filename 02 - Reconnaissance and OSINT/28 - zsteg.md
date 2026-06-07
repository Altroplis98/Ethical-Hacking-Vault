---
tags: [pentest, recon, stego, zsteg, png, both]
tool: zsteg
phase: 1
---
# zsteg

Detect and extract steganographic data hidden in PNG and BMP files using LSB (Least Significant Bit) encoding.

[[02 - Reconnaissance and OSINT/00 - README|Folder index]]

## Install

```bash
gem install zsteg
```

## Usage

```bash
# Auto-detect hidden data
zsteg image.png

# All channels and bit orders
zsteg -a image.png

# Extract specific payload
zsteg -e "b1,rgb,lsb,xy" image.png > extracted.bin

# Check for specific string
zsteg image.png | grep -i "flag\|password\|key"
```

## Common LSB channels

| Channel | Description |
| --- | --- |
| `b1,rgb,lsb,xy` | 1-bit RGB LSB, row by row |
| `b1,r,lsb,xy` | 1-bit Red channel LSB |
| `b1,bgr,lsb,xy` | 1-bit BGR LSB |
| `b2,rgb,lsb,xy` | 2-bit RGB LSB |

## See also

- [[26 - Steghide]] — JPEG steganography
- [[29 - Binwalk]] — embedded file extraction
