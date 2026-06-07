---
tags: [pentest, recon, osint, email, holehe, both]
tool: holehe
phase: 1
---
# holehe

Check if an email address is registered on 120+ websites (Twitter, Instagram, Spotify, etc.) without alerting the target.

[[02 - Reconnaissance and OSINT/00 - README|Folder index]]

## Install

```bash
pip install holehe --break-system-packages
```

## Usage

```bash
# Check a single email
holehe target@example.com

# Output: shows which services have an account with that email
# [+] twitter.com — exists
# [+] instagram.com — exists
# [-] facebook.com — not found
```

## Use cases

| Scenario | Value |
| --- | --- |
| Pre-phishing recon | Know which platforms the target uses |
| Password reuse | If breached on one platform, try creds on others |
| Social engineering | Reference their hobbies/interests from platform profiles |
| Username pivoting | Same email on multiple platforms → same username pattern |

## Limitations

- Rate limiting — some services throttle after many checks
- False negatives — some sites don't reveal registration status
- May trigger login alerts on some platforms

## See also

- [[17 - CrossLinked]] — find employee names for email generation
- [[19 - h8mail]] — find actual breached passwords for these emails
