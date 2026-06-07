---
tags: [pentest, recon, osint, cloud, gcp, buckets, both]
tool: gcpbucketbrute
phase: 1
---
# GCPBucketBrute

Brute-force Google Cloud Storage bucket names and check permissions.

[[02 - Reconnaissance and OSINT/00 - README|Folder index]]

## Install

```bash
git clone https://github.com/RhinoSecurityLabs/GCPBucketBrute.git
cd GCPBucketBrute
pip install -r requirements.txt --break-system-packages
```

## Usage

```bash
# Brute-force bucket names with a keyword
python3 gcpbucketbrute.py -k companyname

# With a specific wordlist
python3 gcpbucketbrute.py -k companyname -w custom_mutations.txt

# Using a service account for authenticated checks
python3 gcpbucketbrute.py -k companyname -f sa_key.json
```

## Permissions checked

| Permission | Risk |
| --- | --- |
| allUsers read | Anyone can list and download files |
| allUsers write | Anyone can upload (malware hosting, defacement) |
| allAuthenticatedUsers read | Any Google account can access |

## See also

- [[33 - cloud_enum]] — multi-cloud enumeration
- [[34 - s3scanner]] — AWS equivalent
- [[36 - MicroBurst Azure]] — Azure equivalent
