---
tags: [pentest, reporting, checklist, post-engagement]
phase: 7
---

# Post-Engagement Checklist

Everything that needs to happen after testing ends and before you close the engagement.

[[08 - Tracks and Reporting/00 - README|Folder index]]

## Checklist

```text
Cleanup:
[ ] Remove all backdoors, shells, and persistence mechanisms
[ ] Remove all uploaded files (web shells, tools, test files)
[ ] Delete any created accounts
[ ] Restore any modified configurations
[ ] Verify cleanup with client IT

Evidence:
[ ] All terminal logs saved and hashed
[ ] Screenshots organized and timestamped
[ ] Evidence encrypted on secure volume
[ ] Raw tool output archived

Report:
[ ] Draft report written
[ ] Internal peer review completed
[ ] Executive summary reviewed for non-technical clarity
[ ] All findings have CVSS scores
[ ] Evidence screenshots embedded
[ ] Attack narrative complete
[ ] Remediation recommendations are specific and actionable

Delivery:
[ ] Draft report sent to client (encrypted)
[ ] Client review period (typically 5-7 business days)
[ ] Address client feedback
[ ] Final report delivered
[ ] Read-out call scheduled

Post-delivery:
[ ] Retest window agreed (if applicable)
[ ] Evidence retention period confirmed
[ ] Evidence destruction scheduled
[ ] Lessons learned documented internally
[ ] Client satisfaction check
```

## See also

- [[14 - Read-Out Call Tips]] — presenting findings to the client
- [[11 - Evidence Handling]] — evidence lifecycle
