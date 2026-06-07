---
tags: [pentest, reporting, cvss, scoring]
phase: 7
---

# CVSS 3.1 and 4.0 Scoring

How to assign CVSS scores to findings. Use the calculator at `https://www.first.org/cvss/calculator/`.

[[08 - Tracks and Reporting/00 - README|Folder index]]

## CVSS 3.1 Base Score metrics

| Metric | Values | Description |
| --- | --- | --- |
| Attack Vector (AV) | N/A/L/P | Network, Adjacent, Local, Physical |
| Attack Complexity (AC) | L/H | Low, High |
| Privileges Required (PR) | N/L/H | None, Low, High |
| User Interaction (UI) | N/R | None, Required |
| Scope (S) | U/C | Unchanged, Changed |
| Confidentiality (C) | N/L/H | None, Low, High |
| Integrity (I) | N/L/H | None, Low, High |
| Availability (A) | N/L/H | None, Low, High |

## Score ranges

| Score | Severity |
| --- | --- |
| 0.0 | None |
| 0.1-3.9 | Low |
| 4.0-6.9 | Medium |
| 7.0-8.9 | High |
| 9.0-10.0 | Critical |

## Common finding scores

| Finding | Typical CVSS | Vector |
| --- | --- | --- |
| Unauthenticated RCE | 9.8 | AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H |
| SQLi with data access | 8.6 | AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:N/A:N |
| Stored XSS | 6.1 | AV:N/AC:L/PR:N/UI:R/S:C/C:L/I:L/A:N |
| Missing security headers | 3.7 | AV:N/AC:H/PR:N/UI:N/S:U/C:L/I:N/A:N |
| Default credentials (internal) | 7.5 | AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N |

## CVSS 4.0 changes

CVSS 4.0 replaces Scope with two new metrics (Subsequent System Impact), adds Attack Requirements, and separates Exploitability from Impact more clearly. Adoption is still growing — check with your client which version they expect.

## See also

- [[08 - Finding Template]] — where CVSS scores go in the report
