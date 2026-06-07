---
tags: [pentest, cheatsheet, cloud, s3, gcs, azure, storage, both]
phase: reference
---
# Cloud Storage (S3 / GCS / Azure Blob)

[[09 - Service Cheatsheets/00 - README|Folder index]]

## AWS S3

```bash
# List bucket contents (if public)
aws s3 ls s3://bucket-name --no-sign-request

# Download
aws s3 cp s3://bucket-name/file.txt . --no-sign-request
aws s3 sync s3://bucket-name/ /tmp/s3dump/ --no-sign-request

# Check permissions
aws s3api get-bucket-acl --bucket bucket-name --no-sign-request
```

## GCP Storage

```bash
# Public bucket
gsutil ls gs://bucket-name
gsutil cp gs://bucket-name/file.txt .

# Without auth
curl https://storage.googleapis.com/bucket-name/
```

## Azure Blob Storage

```bash
# Public container
az storage blob list --container-name container --account-name account --output table
az storage blob download --container-name container --name file.txt --account-name account --file ./file.txt

# Anonymous access
curl "https://account.blob.core.windows.net/container?restype=container&comp=list"
```

## Discovery tools

```bash
cloud_enum -k companyname
s3scanner scan --bucket bucket-name
```

## What to look for

| Content | Impact |
| --- | --- |
| Database backups | Full data breach |
| Config files | API keys, credentials |
| Source code | Application logic, secrets |
| Log files | Session tokens, PII |
| Employee data | PII exposure |

> Full details: [[02 - Reconnaissance and OSINT/33 - cloud_enum|cloud_enum]], [[02 - Reconnaissance and OSINT/34 - s3scanner|s3scanner]]
