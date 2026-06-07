---
tags: [pentest, recon, stego, qr, zbar, both]
tool: zbarimg
phase: 1
---
# zbarimg QR Decode

Command-line QR code and barcode reader. Useful for decoding QR codes found in screenshots, steganography, or documents.

[[02 - Reconnaissance and OSINT/00 - README|Folder index]]

## Install

```bash
sudo apt install zbar-tools -y
```

## Usage

```bash
# Decode QR from image
zbarimg qr_code.png

# Quiet output (data only)
zbarimg -q qr_code.png

# Decode from screenshot
zbarimg screenshot.png
```

## When you encounter QR codes

| Context | Action |
| --- | --- |
| CTF stego challenge | Extract from spectrogram → decode |
| Physical pentest | Photograph QR → decode for URLs/WiFi creds |
| Document forensics | QR in PDFs may contain URLs or data |

## See also

- [[31 - Audacity Spectrogram]] — QR codes sometimes hidden in spectrograms
- [[29 - Binwalk]] — another way to extract hidden data
