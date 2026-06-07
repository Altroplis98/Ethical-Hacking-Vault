---
tags: [pentest, recon, osint, cloud, aws, azure, gcp, both]
tool: cloud_enum
phase: 1
---
# cloud_enum

Multi-cloud enumeration tool. Brute-forces storage buckets, app services, and databases across AWS, Azure, and GCP.

[[02 - Reconnaissance and OSINT/00 - README|Folder index]]

## Install

```bash
git clone https://github.com/initstring/cloud_enum.git
cd cloud_enum
pip install -r requirements.txt --break-system-packages
```

## Usage

```bash
# Enumerate a keyword across all three clouds
python3 cloud_enum.py -k companyname

# With a mutations file
python3 cloud_enum.py -k companyname -m mutations.txt

# Specific cloud only
python3 cloud_enum.py -k companyname --disable-azure --disable-gcp  # AWS only

# Output
python3 cloud_enum.py -k companyname -l results.txt
```

## What it checks

| Cloud | Resources enumerated |
| --- | --- |
| AWS | S3 buckets, EC2 instances (via DNS), AWS apps |
| Azure | Storage blobs, App Services, databases, VMs |
| GCP | Storage buckets, App Engine, Firebase, Cloud Functions |

## Keyword strategy

Use company name, project names, product names, acquisitions:

```text
companyname
company-name
companynamedev
companyname-staging
companyname-backup
companyname-data
projectalpha
```

## See also

- [[34 - s3scanner]] — dedicated AWS S3 scanner
- [[35 - GCPBucketBrute]] — dedicated GCP bucket scanner
- [[36 - MicroBurst Azure]] — Azure-specific enumeration
