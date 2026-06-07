---
tags: [pentest, htb, crypto, steganography, walkthrough-pattern, both]
type: workflow
---
# Crypto / Stego Box - Walkthrough Pattern

[[00 - README|Folder index]]

Less common on standard pentest boxes, but heavily used in HTB Challenges and OSINT/forensics rooms on TryHackMe.

## Hallmarks

- You get a *file* (image, audio, archive, text) and almost no other context
- The "service" is just a download or a static page hosting the artifact

## Universal first pass

```bash
# 1. Identify what you actually have
file mystery
# Don't trust the extension - file checks magic bytes

# 2. Inspect metadata
exiftool mystery

# 3. Look at raw strings
strings -n 8 mystery | head -50
strings -el mystery | head -50         # UTF-16LE (Windows artifacts)

# 4. Hex dump the start and end
xxd mystery | head -20
xxd mystery | tail -20

# 5. Carve embedded data
binwalk mystery
binwalk -e mystery         # extract
foremost -i mystery -o carved/

# 6. Hash and reverse-search the file (sometimes it's a known CTF artifact)
sha256sum mystery
md5sum mystery
```

## If it's an image

```bash
# Standard stego tools
exiftool image.jpg
steghide info image.jpg
steghide extract -sf image.jpg                    # prompts for passphrase
steghide extract -sf image.jpg -p ''              # try empty
stegseek image.jpg /usr/share/wordlists/rockyou.txt  # brute steghide passphrase

# LSB / channel analysis (PNG/BMP)
zsteg -a image.png         # tries every common LSB combo
zsteg --extract b1,r,lsb,xy image.png > out.bin

# Color/channel manipulation - look at each plane
# Use Stegsolve.jar - GUI - cycles channels
java -jar Stegsolve.jar

# Search the image visually for hidden text by inverting / increasing contrast
convert image.jpg -negate inverted.jpg
convert image.jpg -level 0%,50% darker.jpg
```

## If it's audio (WAV / MP3 / FLAC)

```bash
# 1. Open in Audacity
audacity audio.wav

# In Audacity:
# - View > Spectrogram → look for hidden text, QR, or patterns in the spectrum
# - Look at the high-frequency band specifically (>15 kHz, often where data hides)
# - Effect > Reverse → some clues are played backwards
# - Look for DTMF tones (touch-tones spelling out numbers)

# CLI alternatives
sox audio.wav -n spectrogram -o spec.png
sonic-visualiser audio.wav

# Morse code in audio - tool: morse-decoder.com or aurally

# Steghide also supports WAV / AU
steghide extract -sf audio.wav
```

## If it's a QR / barcode

```bash
zbarimg qr.png                  # decodes QR codes
# Web: zxing.org/w/decode
```

## If it's an archive

```bash
# Check for nested archives (matryoshka)
file archive.zip
unzip -l archive.zip
7z l archive.7z

# Password-protected:
zip2john archive.zip > h && john --wordlist=rockyou.txt h
7z2john archive.7z > h && john --wordlist=rockyou.txt h
rar2john archive.rar > h && john --wordlist=rockyou.txt h
```

## If it's "encrypted text" / a cipher

Quick checks before you commit hours:

| Looks like... | Probably is... |
| --- | --- |
| `YWJj`, `ZGVm` (a-z A-Z 0-9 + /=) | Base64. `echo X \| base64 -d` |
| Hex pairs `5f7a9b...` | Hex. `xxd -r -p` |
| All caps, 26 letters, spaces preserved | Substitution cipher. quipqiup.com |
| `Synp` instead of `Flag` | ROT13. `tr 'A-Za-z' 'N-ZA-Mn-za-m'` |
| Random-looking + base64 chars | base64 (try `-d` multiple times) |
| 16/24/32 bytes of "noise" | Probably AES output - need key |
| Starts with `-----BEGIN` | PEM (RSA key, cert, signature) |
| Starts with `gAAAAAB` | Python Fernet |
| Starts with `$argon2`, `$pbkdf2`, `$2b$`, `$6$`, `0x` | Hash. Run `hashid` |

```bash
# Convenient tools
cyberchef                       # web - drop & try multiple decoders
echo "X" | base64 -d            # base64
echo "X" | xxd -r -p            # hex
echo "X" | tr 'A-Za-z' 'N-ZA-Mn-za-m'   # ROT13
hashid '<hash>'                 # identify
```

## RSA tricks worth knowing

```python
# Small e (e=3) + no padding + small message → cube root attack
# Common modulus attack - same N, different e
# Wiener's attack - small d (d < N^0.25)
# Hastad broadcast - same e with N1,N2,N3,...

# Tool: RsaCtfTool
RsaCtfTool --publickey pub.pem --uncipherfile encrypted
```

## Hidden in plain sight

When stuck, recheck:

- HTML comments
- HTTP response headers (Server, X-, custom)
- DNS TXT records of any domain mentioned
- The page's source view vs. rendered view
- Image filenames (sometimes the flag is the filename)
- The EXIF GPS coordinates (geocache style)
- The first letters of each paragraph (acrostic)

> [!tip] If it's not yielding to tools
> Crypto/stego CTFs reward LATERAL THINKING more than tooling. Step back and ask: "What kind of person made this puzzle, and what 'aha' would they have wanted me to feel?"
