---
tags: [pentest, recon, osint, cloud, aws, s3, both]
tool: s3scanner
phase: 1
---
# s3scanner

Scans for open AWS S3 buckets and dumps accessible contents.

[[02 - Reconnaissance and OSINT/00 - README|Folder index]]

## Install

```bash
pip install s3scanner --break-system-packages
```

## Usage

```bash
# Check a list of bucket names
s3scanner scan --buckets-file bucket_names.txt

# Check a single bucket
s3scanner scan --bucket companyname-backup

# Dump accessible contents
s3scanner dump --bucket open-bucket-name --out-dir /tmp/s3dump/
```

## Generating bucket name lists

```bash
# Common patterns
echo "companyname" > buckets.txt
echo "companyname-backup" >> buckets.txt
echo "companyname-dev" >> buckets.txt
echo "companyname-staging" >> buckets.txt
echo "companyname-logs" >> buckets.txt
echo "companyname-data" >> buckets.txt
echo "companyname-assets" >> buckets.txt
echo "companyname-uploads" >> buckets.txt
```

## What to look for in open buckets

| Content | Impact |
| --- | --- |
| Database backups | Full data breach |
| Log files | Credentials, session tokens |
| Config files | API keys, database connection strings |
| Source code | Application logic, hardcoded secrets |
| Employee data | PII exposure |

> [!danger] Don't download everything
> In a pentest, document the exposure and sample a few files for PoC. Bulk downloading PII exceeds most ROE.

## See also

- [[33 - cloud_enum]] — multi-cloud enumeration
- [[35 - GCPBucketBrute]] — GCP equivalent
