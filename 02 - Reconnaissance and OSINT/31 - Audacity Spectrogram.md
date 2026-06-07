---
tags: [pentest, recon, stego, audio, audacity, both]
tool: audacity
phase: 1
---
# Audacity Spectrogram

Hidden messages can be encoded in audio files as visual patterns in the spectrogram view. Common in CTFs.

[[02 - Reconnaissance and OSINT/00 - README|Folder index]]

## How to check

1. Open the audio file in Audacity
2. Click the track name dropdown → **Spectrogram**
3. Look for visible text, QR codes, or patterns in the frequency display

## Spectrogram settings for best visibility

- Track dropdown → Spectrogram Settings
- Scale: Logarithmic (for text) or Linear (for images)
- Window size: 2048 or 4096 (larger = better frequency resolution)
- Min/Max frequency: adjust to zoom into the relevant range

## Other audio stego checks

```bash
# Check file for strings
strings audio.wav | head -50

# Check with binwalk for embedded files
binwalk audio.wav

# Check with steghide (WAV supported)
steghide info audio.wav
steghide extract -sf audio.wav -p ""

# Sonic Visualiser (alternative to Audacity)
# Offers more spectrogram analysis options
```

## SSTV (Slow Scan Television)

Some CTFs encode images as SSTV signals in audio:

```bash
# Decode SSTV
# Use qsstv on Linux or Robot36 on Android
sudo apt install qsstv
```

## See also

- [[26 - Steghide]] — also supports WAV files
- [[32 - zbarimg QR Decode]] — if spectrogram reveals a QR code
