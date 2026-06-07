---
tags: [pentest, pre-engagement, communications, reporting]
phase: 0
---

# Communications Plan

Who you call, when you call them, and what you say. Miscommunication during an engagement can cause panic, legal issues, or missed critical findings.

[[01 - Pre-Engagement/00 - README|Folder index]]

## Contact matrix

```text
Role                Name            Phone           Email               Signal
─────────────────────────────────────────────────────────────────────────────
Client POC          [Name]          [Phone]         [Email]             [handle]
Client IT Lead      [Name]          [Phone]         [Email]             [handle]
Client CISO         [Name]          [Phone]         [Email]             —
Tester Lead         [You]           [Phone]         [Email]             [handle]
Tester Backup       [Name]          [Phone]         [Email]             [handle]
Legal (your firm)   [Name]          [Phone]         [Email]             —
```

> [!warning] Fill this out BEFORE the engagement starts
> At 2am when you find a critical vuln, you don't want to be searching for phone numbers.

## Communication triggers

| Event | Who to notify | Channel | Deadline |
| --- | --- | --- | --- |
| Engagement start | Client POC | Email | Day of |
| Critical finding (RCE, DA, data breach) | Client POC + CISO | Signal → Phone if no response in 30min | Within 1 hour |
| Accidental service disruption | Client IT Lead | Phone immediately | Immediate |
| PII / PHI discovered | Client POC + Legal | Encrypted email | Within 4 hours |
| Scope question (is X in scope?) | Client POC | Signal | Before testing X |
| Engagement complete | Client POC | Email | End of testing window |
| Draft report ready | Client POC | Email with encrypted attachment | Per SOW timeline |

## Status updates

```text
Frequency: Daily at 17:00 EST (or as agreed)
Format:    Encrypted email or Signal message
Content:
  - Systems tested today
  - Findings (severity + one-liner, no technical detail over email)
  - Planned targets for tomorrow
  - Any blockers or scope questions
```

### Example daily update

```text
Subject: [ENGAGEMENT] Daily Update - Day 3

Systems tested: 10.10.10.0/24 (web servers), app.client.com
Findings today:
  - 1x Critical: Unauthenticated RCE on 10.10.10.15 (already notified via Signal)
  - 2x High: SQL injection on staging app
  - 1x Medium: Default credentials on internal Jenkins

Tomorrow: Moving to internal AD enumeration (10.10.20.0/24)
Blockers: None

— [Your name], [Firm]
```

## Escalation path

```text
Level 1: Signal message to Client POC
  ↓ No response in 30 minutes
Level 2: Phone call to Client POC
  ↓ No response in 15 minutes
Level 3: Phone call to Client IT Lead
  ↓ No response in 15 minutes
Level 4: Phone call to Client CISO
  ↓ No response
Level 5: Email to all three + your firm's legal
```

## Encrypted communication setup

```bash
# PGP key exchange (do this during kickoff)
gpg --gen-key
gpg --export --armor your@email.com > your_pubkey.asc
# Send to client, import theirs:
gpg --import client_pubkey.asc

# Encrypting a file for the client
gpg --encrypt --recipient client@company.com report_draft.pdf
```

## Deconfliction

If the client's SOC detects your activity and escalates it as a real incident:

1. **Have a deconfliction code** — a pre-agreed phrase that identifies you
2. **Source IPs documented** — your testing IPs should be whitelisted or at least known
3. **Time window alignment** — SOC should know your testing window

```text
Deconfliction code: "TANGO-FOXTROT-2024"
Tester source IPs: 198.51.100.10, 198.51.100.11
```

> [!tip] The SOC should be told testing is happening
> Whether they're told WHO is testing and WHAT the scope is depends on the engagement type (announced vs. unannounced red team).

## See also

- [[02 - Rules of Engagement]] — the tactical rules this plan supports
- [[06 - Operational Logging]] — what you log alongside these communications
