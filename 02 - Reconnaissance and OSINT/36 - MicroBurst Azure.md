---
tags: [pentest, recon, osint, cloud, azure, microburst, both]
tool: microburst
phase: 1
---
# MicroBurst Azure

PowerShell toolkit for Azure security assessment. Enumerates storage accounts, VMs, web apps, and more.

[[02 - Reconnaissance and OSINT/00 - README|Folder index]]

## Install

```powershell
# PowerShell
Install-Module -Name MicroBurst -Scope CurrentUser
Import-Module MicroBurst
```

## Key functions

### Storage account enumeration (unauthenticated)

```powershell
# Brute-force storage account names
Invoke-EnumerateAzureBlobs -Base companyname

# Check specific blob containers
Invoke-EnumerateAzureBlobs -Base companyname -Permutations permutations.txt
```

### Subdomain enumeration

```powershell
# Find Azure-hosted services
Invoke-EnumerateAzureSubDomains -Base companyname -Verbose
```

### Authenticated enumeration (post-access)

```powershell
# After authenticating to Azure
Get-AzDomainInfo -Verbose

# Enumerate VMs and their configs
Get-AzVMInfo

# Find automation account credentials
Get-AzAutomationCredential
```

## What it enumerates

| Resource type | Unauthenticated | Authenticated |
| --- | --- | --- |
| Storage blobs | Yes | Yes |
| Web apps | Yes (subdomain brute) | Yes |
| VMs | No | Yes |
| Key Vault | No | Yes |
| Automation accounts | No | Yes |
| SQL databases | Yes (subdomain) | Yes |

## Linux alternative (CLI)

```bash
# Azure CLI for basic enumeration
az storage account list
az webapp list
az vm list
az keyvault list
```

## See also

- [[33 - cloud_enum]] — multi-cloud alternative (Python, Linux-friendly)
- [[34 - s3scanner]] — AWS equivalent
