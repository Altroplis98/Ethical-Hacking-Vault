---
tags: [pentest, recon, stego, steghide, both]
tool: steghide
phase: 1
---
# Steghide

Embeds and extracts hidden data in JPEG, BMP, WAV, and AU files. The most common steganography tool in CTFs and some real-world scenarios.

[[02 - Reconnaissance and OSINT/00 - README|Folder index]]

## Install / verify

```bash
which steghide || sudo apt install steghide -y
```

## Usage

```bash
# Extract hidden data (will prompt for passphrase)
steghide extract -sf image.jpg

# Extract with empty passphrase
steghide extract -sf image.jpg -p ""

# Extract with known passphrase
steghide extract -sf image.jpg -p "secret"

# Check if file has embedded data
steghide info image.jpg

# Embed data (for testing)
steghide embed -cf cover.jpg -ef secret.txt -p "password"
```

## Brute-force passphrase

```bash
# Use stegseek (much faster than manual brute-force)
stegseek image.jpg /usr/share/wordlists/rockyou.txt
```

## Supported formats

| Format | Extensions |
| --- | --- |
| JPEG | .jpg, .jpeg |
| BMP | .bmp |
| WAV | .wav |
| AU | .au |

> [!tip] If steghide doesn't work, the data might be hidden with a different tool
> Try zsteg (PNG), binwalk (embedded files), or check for LSB encoding.

## See also

- [[27 - Stegseek]] — brute-force steghide passphrases
- [[28 - zsteg]] — PNG steganography
- [[29 - Binwalk]] — embedded file extraction
