---
tags: [pentest, recon, osint, moc, both]
phase: 1
---
# 02 - Reconnaissance and OSINT

Passive (no packets to target) and active (light interaction) information gathering.

[[00 - Vault Index|Home]] · Prev: [[01 - Pre-Engagement/00 - README|Pre-Engagement]] · Next: [[03 - Scanning/00 - README|Scanning]]

## Files in this folder

### Domain & DNS recon
- [[01 - WHOIS]]
- [[02 - DNS Recon Basics]]
- [[03 - Certificate Transparency]]
- [[04 - Subfinder]]
- [[05 - Amass]]
- [[06 - Assetfinder]]
- [[07 - Findomain]]
- [[08 - Sublist3r]]
- [[09 - dnsenum dnsrecon fierce]]

### Search-engine + framework OSINT
- [[10 - theHarvester]]
- [[11 - Recon-ng]]
- [[12 - Spiderfoot]]
- [[13 - Maltego]]
- [[14 - Google Dorking]]
- [[15 - Shodan]]
- [[16 - Censys]]

### People & email
- [[17 - CrossLinked]]
- [[18 - holehe]]
- [[19 - h8mail]]

### Historical content + secrets
- [[20 - Wayback Machine]]
- [[21 - gau and katana]]
- [[22 - Gitleaks and TruffleHog]]

### Document metadata
- [[23 - Metagoofil]]
- [[24 - ExifTool]]

### Image OSINT & steganography
- [[25 - Image Geolocation]]
- [[26 - Steghide]]
- [[27 - Stegseek]]
- [[28 - zsteg]]
- [[29 - Binwalk]]
- [[30 - Foremost]]
- [[31 - Audacity Spectrogram]]
- [[32 - zbarimg QR Decode]]

### Cloud asset discovery
- [[33 - cloud_enum]]
- [[34 - s3scanner]]
- [[35 - GCPBucketBrute]]
- [[36 - MicroBurst Azure]]

## Methodology

```text
1. Confirm scope (domain list, IP ranges)
2. WHOIS + DNS basics → identify name servers, mail, NS records
3. Subdomain sweep (passive first: subfinder, amass passive, CT logs)
4. Active DNS brute only if scope allows
5. theHarvester / spiderfoot for emails + creds + social
6. Google dorks for leaked docs
7. Shodan / Censys for exposed services
8. GitHub / wayback for leaked secrets
9. Cloud bucket discovery
10. Geolocate / metadata-strip any images you find
```

> [!tip] Order matters
> Passive first, active later. The longer you stay invisible, the more value you provide.
