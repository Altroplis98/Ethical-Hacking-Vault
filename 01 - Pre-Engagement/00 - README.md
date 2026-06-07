---
tags: [pentest, pre-engagement, moc]
phase: 0
---

# 01 - Pre-Engagement

The legal and procedural foundation. Skipping this turns ethical hacking into criminal hacking.

[[00 - Vault Index|Home]] · Next: [[02 - Reconnaissance and OSINT/00 - README|Recon & OSINT]]

## Files in this folder

- [[01 - NDA and SOW]] - non-disclosure agreement, statement of work
- [[02 - Rules of Engagement]] - allowed techniques, windows, data handling
- [[03 - Scope Definition]] - in/out of scope, third-party authorization
- [[04 - Cloud Testing Policies]] - AWS/Azure/GCP/M365 specific rules
- [[05 - Attack Infrastructure Setup]] - Kali update, wordlists, VPN, Tor
- [[06 - Operational Logging]] - capturing every command for evidence chain
- [[07 - Communications Plan]] - contacts, escalation, status cadence

## Decision flow

```text
RFP / inquiry
    ↓
Scoping call → produce SOW draft
    ↓
NDA signed → ROE drafted
    ↓
Auth letter + 3rd-party consent obtained
    ↓
Infra built, wordlists synced, evidence dir created
    ↓
Kick-off call → GO
```

> [!warning] If you can't produce the signed scope + ROE + auth letter in 30 seconds, the engagement does not start.
