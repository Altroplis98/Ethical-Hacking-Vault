---
tags: [pentest, recon, osint, geolocation, images, both]
phase: 1
---
# Image Geolocation

Extract or estimate physical locations from images using EXIF GPS data, visual landmarks, and OSINT techniques.

[[02 - Reconnaissance and OSINT/00 - README|Folder index]]

## EXIF GPS extraction

```bash
# Check for GPS data
exiftool -GPSLatitude -GPSLongitude -GPSPosition photo.jpg

# Convert to decimal degrees
exiftool -n -GPSLatitude -GPSLongitude photo.jpg

# Batch extraction
exiftool -csv -GPSLatitude -GPSLongitude -r /path/to/photos/ > gps_data.csv
```

## Online geolocation tools

| Tool | Use case |
| --- | --- |
| Google Maps | Paste coordinates from EXIF |
| GeoGuessr (paid) | Practice visual geolocation skills |
| Yandex Reverse Image Search | Often better than Google for buildings/locations |
| Google Lens | Identify landmarks in photos |
| SunCalc | Estimate time from shadows |
| What3words | Precise location sharing |

## Visual geolocation (no EXIF)

When EXIF is stripped, look for:

| Clue | Method |
| --- | --- |
| Street signs | Language, format, naming conventions |
| License plates | Country/state format |
| Vegetation | Climate zone estimation |
| Sun position/shadows | Hemisphere + approximate time |
| Architecture | Regional building styles |
| Power lines/poles | Different by country |
| Road markings | Line color/style varies by country |
| Store signs / brands | Region-specific chains |

## See also

- [[24 - ExifTool]] — the tool for EXIF extraction
- [[26 - Steghide]] — images may also contain hidden data
