---
tags: [pentest, pre-engagement, legal, nda, sow]
phase: 0
---

# NDA and SOW

The Non-Disclosure Agreement and Statement of Work are the two documents that define what you're doing and what you can't talk about.

[[01 - Pre-Engagement/00 - README|Folder index]]

## NDA (Non-Disclosure Agreement)

Protects both sides. You'll see client data; they need assurance you won't leak it.

### Key clauses to confirm

| Clause | Why it matters |
| --- | --- |
| Scope of confidential info | Defines what you can't disclose — screenshots, credentials, findings |
| Duration | Typically 2-5 years; some are perpetual |
| Permitted disclosures | Your team, subcontractors, legal counsel |
| Return / destruction | What happens to loot files, notes, VM snapshots post-engagement |
| Carve-outs | Public info, independently discovered info, legally compelled disclosure |

### Mutual vs. one-way

- **Mutual NDA** — both parties protect each other's info. Preferred.
- **One-way NDA** — only you are bound. Common when the client is much larger.

> [!tip] Sign the NDA before the scoping call
> Clients sometimes share internal architecture during scoping. Protect yourself.

## SOW (Statement of Work)

The contract that says exactly what you'll deliver, when, and for how much.

### SOW template sections

```text
1. Engagement overview        — one paragraph summary
2. Scope                      — targets, networks, applications, exclusions
3. Methodology                — PTES, OWASP, NIST 800-115, custom
4. Timeline                   — start, end, milestone dates
5. Deliverables               — report format, debrief call, retest window
6. Assumptions & constraints  — VPN access provided, credentials for auth testing, etc.
7. Pricing & payment terms    — fixed fee, T&M, milestone billing
8. Out-of-scope               — explicitly list what you will NOT test
9. Change control             — how scope changes are handled mid-engagement
10. Signatures                — both parties, dated
```

### Common mistakes

| Mistake | Impact |
| --- | --- |
| Vague scope ("test everything") | Scope creep; arguments over what's covered |
| No exclusion list | You accidentally DoS a prod system nobody mentioned |
| Missing retest clause | Client expects free retesting; you expected billable hours |
| No emergency contact | You find critical RCE at 2am with nobody to call |

## See also

- [[02 - Rules of Engagement]] — the tactical companion to the SOW
- [[03 - Scope Definition]] — detailed scoping guidance
