---
tags: [pentest, recon, stego, foremost, forensics, both]
tool: foremost
phase: 1
---
# Foremost

File carving tool — recovers files from disk images or binary blobs based on file headers, footers, and internal structures.

[[02 - Reconnaissance and OSINT/00 - README|Folder index]]

## Install / verify

```bash
which foremost || sudo apt install foremost -y
```

## Usage

```bash
# Carve all supported file types
foremost -i suspicious_file.bin -o /tmp/carved/

# Specific file types
foremost -t jpg,png,pdf,zip -i disk_image.dd -o /tmp/carved/

# Verbose
foremost -v -i suspicious_file.bin -o /tmp/carved/
```

## Supported file types

`jpg, gif, png, bmp, avi, exe, mpg, wav, riff, wmv, mov, pdf, ole, doc, zip, rar, htm, cpp, all`

## When to use foremost vs. binwalk

| Tool | Best for |
| --- | --- |
| binwalk | Firmware, embedded archives, file signatures within other files |
| foremost | Disk images, raw data carving, deleted file recovery |

## See also

- [[29 - Binwalk]] — embedded file extraction
- [[31 - Audacity Spectrogram]] — audio steganography
