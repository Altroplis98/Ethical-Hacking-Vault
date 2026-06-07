---
tags: [pentest, pre-engagement, roe, legal]
phase: 0
---

# Rules of Engagement

The ROE is the tactical rulebook — what you can do, when, how hard, and what's off limits.

[[01 - Pre-Engagement/00 - README|Folder index]]

## ROE components

### Testing window

```text
Primary window:   Mon-Fri 22:00 - 06:00 EST
Fallback window:  Sat 08:00 - Sun 20:00 EST (requires 24h notice)
Blackout dates:   2024-12-20 through 2025-01-03 (code freeze)
```

Define this explicitly. "Business hours" means different things to different people.

### Allowed techniques

| Category | Allowed | Restricted | Prohibited |
| --- | --- | --- | --- |
| Network scanning | Full port scans | Aggressive timing (T5) | DoS / stress testing |
| Brute force | Targeted spray (5 attempts/account) | Full dictionary | Account lockout attacks |
| Social engineering | Phishing (email only) | Vishing | Physical intrusion |
| Exploitation | Known CVEs, misconfigs | Custom 0-day | Destructive payloads |
| Data exfil | PoC (10 records max) | Full DB dump | PII/PHI extraction |

### Notification thresholds

| Finding severity | Action |
| --- | --- |
| Critical (RCE on perimeter, DA compromise) | Notify client POC within 1 hour |
| Service disruption (accidental) | Notify immediately + stop testing that target |
| PII/PHI discovered | Document, do NOT exfiltrate, notify within 4 hours |

### Communication channels

```text
Primary:    Signal group (tester + client POC + client IT lead)
Backup:     Encrypted email (PGP)
Emergency:  Phone call to client POC → escalate to CISO if unreachable
```

### Evidence handling

- All evidence stored on encrypted volume (LUKS or VeraCrypt)
- Screenshots redact PII where possible
- Evidence retained for 90 days post-report delivery, then destroyed
- Client receives SHA256 hashes of all evidence files

## ROE sign-off

Both the tester lead and the client authorizer sign the ROE. Without this signature, you don't touch a single packet.

```text
Tester Lead:   ________________________  Date: ________
Client Auth:   ________________________  Date: ________
```

> [!warning] "Verbal approval" is not approval
> If it's not signed, it doesn't exist. This protects YOU.

## See also

- [[01 - NDA and SOW]] — the legal wrapper
- [[03 - Scope Definition]] — what's in and out
- [[07 - Communications Plan]] — detailed comms procedures
