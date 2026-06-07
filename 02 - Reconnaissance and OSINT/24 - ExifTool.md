---
tags: [pentest, recon, osint, exiftool, metadata, both]
tool: exiftool
phase: 1
---
# ExifTool

Read, write, and edit metadata in files — images, documents, audio, video. The most comprehensive metadata tool available.

[[02 - Reconnaissance and OSINT/00 - README|Folder index]]

## Install / verify

```bash
which exiftool || sudo apt install libimage-exiftool-perl -y
```

## Usage

```bash
# View all metadata
exiftool document.pdf
exiftool photo.jpg

# Recursive directory scan
exiftool -r /path/to/files/

# Extract specific fields
exiftool -Author -Creator -Producer -Company document.docx

# GPS coordinates from photos
exiftool -GPSLatitude -GPSLongitude photo.jpg

# Export to CSV
exiftool -csv -r /path/to/files/ > metadata.csv

# Strip all metadata (defensive)
exiftool -all= photo.jpg
exiftool -all= -r /path/to/files/  # recursive
```

## Key fields for pentesters

| Field | Where found | Intelligence |
| --- | --- | --- |
| Author / Creator | Documents | Employee names |
| Software | Documents | Application + version |
| Company | Documents | Organization name |
| GPS Position | Images | Physical location |
| Camera Model | Images | Device identification |
| File Path | Documents | Internal directory structure |
| Modify Date | All | Timeline correlation |
| Producer | PDFs | PDF generator software |

## Batch username extraction

```bash
# Extract all author names from downloaded documents
exiftool -r -Author -Creator /tmp/downloaded_docs/ 2>/dev/null | \
  grep -v "^=" | awk -F': ' '{print $2}' | sort -u > authors.txt
```

## See also

- [[23 - Metagoofil]] — automates document download + metadata extraction
- [[25 - Image Geolocation]] — using GPS data from images
