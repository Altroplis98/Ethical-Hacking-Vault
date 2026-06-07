---
tags: [pentest, reporting, evidence, chain-of-custody]
phase: 7
---

# Evidence Handling

Proper evidence collection, storage, and destruction procedures.

[[08 - Tracks and Reporting/00 - README|Folder index]]

## During the engagement

```text
All evidence stored on:
  - Encrypted volume (LUKS on Linux, VeraCrypt on Windows)
  - Dedicated engagement directory structure
  - Timestamped filenames for all screenshots
  - Terminal sessions recorded with `script` or tmux logging
```

## Evidence types

| Type | Examples | Handling |
| --- | --- | --- |
| Screenshots | Proof of access, data exposure | Timestamp, redact PII |
| Command output | Tool results, hash dumps | Save raw + annotated |
| Files obtained | Config files, credentials | Encrypted storage only |
| Network captures | PCAPs (if captured) | Encrypted, delete after report |
| Hashes/credentials | SAM dump, NTDS extract | Encrypted, never sent in cleartext |

## SHA256 hashing

```bash
# Hash all evidence files for integrity
find evidence/ -type f -exec sha256sum {} \; > evidence_hashes.txt
```

## Post-engagement

```text
Timeline:
  Day 0:    Engagement ends
  Day +7:   Draft report delivered
  Day +14:  Final report after client review
  Day +90:  All evidence securely destroyed
            (or per client contract)

Destruction method:
  - Encrypted volumes: delete encryption keys
  - Physical media: NIST 800-88 compliant wipe
  - Cloud storage: delete + verify no backups
  - Document destruction confirmation to client
```

## See also

- [[06 - Report Structure]] — where evidence appears in the report
- [[01 - Pre-Engagement/06 - Operational Logging|Operational Logging]] — capturing evidence during testing
