---
tags: [pentest, enumeration, web, drupal, droopescan, recon]
tool: droopescan
phase: 3
---
# Droopescan

Scanner for Drupal, SilverStripe, WordPress, and Joomla. Identifies versions and known vulnerabilities.

[[04 - Enumeration/00 - README|Folder index]]

## Install

```bash
pip install droopescan --break-system-packages
```

## Usage

```bash
# Scan Drupal site
droopescan scan drupal -u http://10.10.10.10

# Scan SilverStripe
droopescan scan silverstripe -u http://10.10.10.10

# Auto-detect CMS
droopescan scan -u http://10.10.10.10
```

## See also

- [[28 - WPScan]] — WordPress-specific (more thorough for WP)
- [[29 - JoomScan]] — Joomla-specific
