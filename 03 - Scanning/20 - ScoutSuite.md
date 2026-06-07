---
tags: [pentest, scanning, cloud, scoutsuite, aws, azure, gcp, both, recon]
tool: scoutsuite
phase: 2
---
# ScoutSuite

Multi-cloud security auditing tool. Pulls configuration data from AWS, Azure, GCP, and generates an HTML report highlighting misconfigurations.

[[03 - Scanning/00 - README|Folder index]]

## Install

```bash
pip install scoutsuite --break-system-packages
```

## Usage

```bash
# AWS (uses default credentials / profile)
scout aws

# AWS with specific profile
scout aws --profile client-profile

# Azure
scout azure --cli  # uses az CLI auth

# GCP
scout gcp --user-account  # uses gcloud auth
scout gcp --service-account /path/to/key.json

# Output directory
scout aws --report-dir /tmp/scout_report/
```

## What it checks

| Cloud | Checks |
| --- | --- |
| AWS | S3 permissions, security groups, IAM policies, CloudTrail, GuardDuty, KMS, RDS, EC2, Lambda, VPC |
| Azure | Storage accounts, NSGs, Key Vault, App Services, SQL, RBAC, Activity Log |
| GCP | IAM, Compute, Storage, Networking, Logging, BigQuery |

## Report navigation

The HTML report organizes findings by:
1. **Service** (S3, EC2, IAM, etc.)
2. **Severity** (danger, warning, info)
3. **Rule** (specific misconfiguration)

## Common high-severity findings

| Finding | Risk |
| --- | --- |
| S3 bucket publicly accessible | Data exposure |
| Security group allows 0.0.0.0/0 on SSH/RDP | Unrestricted access |
| CloudTrail disabled | No audit trail |
| Root account used without MFA | Account takeover |
| IAM policy with `*:*` | Over-permissioned |

## See also

- [[21 - Prowler AWS]] — AWS-focused with CIS benchmark checks
- [[22 - ROADrecon Azure]] — Azure AD-specific enumeration
