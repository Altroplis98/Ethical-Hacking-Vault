---
tags: [pentest, reporting, narrative, attack-chain]
phase: 7
---

# Attack Narrative

Telling the story of how you went from external/unauthenticated to domain admin (or whatever the objective was). The most impactful part of the report.

[[08 - Tracks and Reporting/00 - README|Folder index]]

## Structure

```text
1. Initial access — how you got your first foothold
2. Enumeration — what you discovered from that position
3. Privilege escalation — how you went from low to high privilege
4. Lateral movement — how you moved between systems
5. Objective achieved — DA, data access, whatever the goal was
```

## Writing tips

- **Chronological** — tell it as a story, step by step
- **Include timestamps** — "At 14:32 UTC on Day 2, we discovered..."
- **Show the chain** — make it clear how finding A led to B led to C
- **Business impact at each step** — not just "got DA" but "accessed all employee salary data in HR share"
- **Include dead ends** — "We attempted X but the client's Y control prevented it" (this shows value of their defenses)
- **Screenshots at key moments** — before/after privilege changes

## Example narrative excerpt

```text
On Day 1, external scanning identified an Apache 2.4.49 server on
10.10.10.15. Using CVE-2021-41773, we achieved remote code execution
as www-data (F-001).

From this foothold, we discovered credentials for the 'svc_backup'
account in a configuration file at /opt/app/config.yml (F-002). This
service account had local admin rights on the backup server
(10.10.10.20).

Using the svc_backup credentials, we authenticated to the backup
server via WinRM and discovered it had unconstrained delegation
enabled (F-003). By coercing authentication from the domain
controller, we captured a TGT for DC01$ and achieved Domain Admin
access within 4 hours of initial access.
```

## See also

- [[08 - Finding Template]] — detailed per-finding documentation
- [[11 - Evidence Handling]] — managing the evidence that supports the narrative
