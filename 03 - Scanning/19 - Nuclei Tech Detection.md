---
tags: [pentest, scanning, nuclei, tech-detect, fingerprint, both, recon, web]
tool: nuclei
phase: 2
---
# Nuclei Tech Detection

Using Nuclei's technology detection templates to fingerprint web technologies at scale. Nuclei goes far beyond detection — it's covered more in [[05 - Vulnerability Analysis/04 - Nuclei Vuln Scanning|Vuln Analysis]] — but its tech detection templates belong in the scanning phase.

[[03 - Scanning/00 - README|Folder index]]

## Install

```bash
go install -v github.com/projectdiscovery/nuclei/v3/cmd/nuclei@latest
nuclei -update-templates
```

## Tech detection templates

```bash
# Run tech detection templates
nuclei -u https://example.com -t technologies/

# Against a list of targets
nuclei -l targets.txt -t technologies/

# Specific tech categories
nuclei -u https://example.com -t technologies/ -tags wordpress
nuclei -u https://example.com -t technologies/ -tags cms

# Combined with httpx pipeline
subfinder -d example.com -silent | httpx -silent | nuclei -t technologies/
```

## What tech templates detect

- CMS platforms (WordPress, Joomla, Drupal, etc.)
- Web frameworks (Laravel, Django, Rails, Spring, etc.)
- JavaScript frameworks (React, Angular, Vue, etc.)
- Server software (Apache, Nginx, IIS versions)
- CDNs and WAFs
- Analytics and tracking
- API technologies

## From detection to exploitation

```bash
# Detect tech → scan for vulns in detected tech
nuclei -u https://example.com -t technologies/ -o tech_results.txt
# Review results, then:
nuclei -u https://example.com -tags wordpress  # if WordPress detected
nuclei -u https://example.com -tags apache     # if Apache detected
```

## See also

- [[05 - Vulnerability Analysis/04 - Nuclei Vuln Scanning|Nuclei Vuln Scanning]] — full vulnerability scanning with Nuclei
- [[18 - httpx]] — HTTP probing that feeds into Nuclei
