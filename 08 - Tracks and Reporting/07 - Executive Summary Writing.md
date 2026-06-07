---
tags: [pentest, reporting, executive-summary]
phase: 7
---

# Executive Summary Writing

The most important page in the report. The CISO reads this; the board might too.

[[08 - Tracks and Reporting/00 - README|Folder index]]

## Structure

```text
Paragraph 1: What we did
  "[Firm] performed a [type] penetration test of [client]'s
   [scope] between [dates]. The assessment followed [methodology]."

Paragraph 2: Overall posture
  "The overall security posture was assessed as [rating].
   [X] critical, [Y] high, [Z] medium findings were identified."

Paragraph 3: Key findings (business language, not technical)
  "An attacker on the corporate network could escalate from
   a standard user account to full domain administrator access
   within [time], enabling access to all corporate data."

Paragraph 4: Strategic recommendations
  "We recommend prioritizing: (1) ..., (2) ..., (3) ..."
```

## Rules

- **No jargon** — "gained admin access" not "obtained DA via Kerberoasting + DCSync chain"
- **Business impact** — "access to all employee payroll data" not "dumped NTDS.dit"
- **1-2 pages max** — if a C-level can't read it in 5 minutes, it's too long
- **Lead with the worst news** — don't bury critical findings
- **Include positives** — "MFA was properly enforced on VPN access"

## Risk ratings

| Rating | Meaning |
| --- | --- |
| Critical | External attacker can compromise the environment with minimal effort |
| High | Significant vulnerabilities exist that could lead to compromise |
| Moderate | Some vulnerabilities exist but exploitation requires specific conditions |
| Low | Minor issues that represent defense-in-depth gaps |

## See also

- [[06 - Report Structure]] — full report layout
- [[08 - Finding Template]] — individual finding format
