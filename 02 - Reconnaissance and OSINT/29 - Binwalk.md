---
tags: [pentest, recon, stego, binwalk, forensics, both]
tool: binwalk
phase: 1
---
# Binwalk

Firmware analysis and embedded file extraction tool. Searches binary files for embedded files, compressed archives, and file system images.

[[02 - Reconnaissance and OSINT/00 - README|Folder index]]

## Install / verify

```bash
which binwalk || sudo apt install binwalk -y
```

## Usage

```bash
# Scan for embedded files
binwalk suspicious_file.png

# Extract embedded files
binwalk -e suspicious_file.png

# Recursive extraction (extract within extracted files)
binwalk -eM suspicious_file.png

# Entropy analysis (detect encrypted/compressed sections)
binwalk -E suspicious_file.png

# Specify extraction directory
binwalk -e -C /tmp/extracted suspicious_file.png
```

## What binwalk finds

| Signature | Examples |
| --- | --- |
| Compressed archives | zip, gzip, bzip2, lzma, xz |
| File systems | squashfs, cramfs, ext, JFFS2 |
| Images | PNG, JPEG, GIF embedded within other files |
| Certificates | X.509 certs, PEM keys |
| Executable code | ELF, PE headers |
| Raw data | Base64, hex-encoded blobs |

## Common CTF pattern

```bash
# Image file that's suspiciously large
file image.png           # confirms it's a PNG
binwalk image.png        # reveals embedded ZIP
binwalk -e image.png     # extracts the ZIP
cd _image.png.extracted/
unzip *.zip              # find the flag
```

## See also

- [[30 - Foremost]] — carve files based on headers/footers
- [[26 - Steghide]] — JPEG steganography
