---
tags: [pentest, recon, stego, stegseek, both]
tool: stegseek
phase: 1
---
# Stegseek

Lightning-fast steghide passphrase cracker. Cracks steghide-embedded data in seconds using wordlists.

[[02 - Reconnaissance and OSINT/00 - README|Folder index]]

## Install

```bash
# Download from GitHub releases
wget https://github.com/RickdeJager/stegseek/releases/latest/download/stegseek_amd64.deb
sudo dpkg -i stegseek_amd64.deb
```

## Usage

```bash
# Crack with rockyou
stegseek image.jpg /usr/share/wordlists/rockyou.txt

# Specify output file
stegseek image.jpg /usr/share/wordlists/rockyou.txt -xf extracted.txt

# Seed-based crack (try without wordlist — tests empty passphrase and common patterns)
stegseek --seed image.jpg
```

## Speed

Stegseek processes **millions of passphrases per second** — it can crack rockyou.txt against a steghide file in under 2 seconds on modern hardware.

## See also

- [[26 - Steghide]] — the tool whose passphrases stegseek cracks
- [[28 - zsteg]] — PNG steganography (different tool entirely)
