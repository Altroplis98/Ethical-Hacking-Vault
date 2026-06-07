---
tags: [pentest, scanning, cloud, prowler, aws, both, recon]
tool: prowler
phase: 2
---
# Prowler AWS

AWS security assessment tool focused on CIS benchmarks, GDPR, HIPAA, and AWS best practices. More AWS-specific depth than ScoutSuite.

[[03 - Scanning/00 - README|Folder index]]

## Install

```bash
pip install prowler --break-system-packages
```

## Usage

```bash
# Full scan with default credentials
prowler aws

# Specific checks
prowler aws --checks s3_bucket_public_access ec2_security_group_open

# CIS benchmark
prowler aws --compliance cis_2.0_aws

# Specific region
prowler aws --region us-east-1

# Output
prowler aws -M csv json html
prowler aws --output-directory /tmp/prowler/
```

## Key compliance frameworks

```bash
prowler aws --compliance cis_2.0_aws       # CIS AWS Foundations
prowler aws --compliance gdpr_aws          # GDPR
prowler aws --compliance hipaa_aws         # HIPAA
prowler aws --compliance pci_3.2.1_aws     # PCI DSS
prowler aws --compliance aws_well_architected_framework  # AWS WAF
```

## High-value checks

| Check | What it finds |
| --- | --- |
| `iam_root_mfa_enabled` | Root account without MFA |
| `s3_bucket_public_access` | Publicly accessible S3 buckets |
| `cloudtrail_multi_region_enabled` | Missing audit logging |
| `ec2_security_group_open` | Unrestricted security groups |
| `rds_instance_public_access` | Databases with public endpoints |

## See also

- [[20 - ScoutSuite]] — multi-cloud alternative
- [[22 - ROADrecon Azure]] — Azure-specific tool
