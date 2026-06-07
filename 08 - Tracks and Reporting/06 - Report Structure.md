---
tags: [pentest, reporting, structure, deliverable]
phase: 7
---

# Report Structure

The standard pentest report layout that clients expect. This is your primary deliverable.

[[08 - Tracks and Reporting/00 - README|Folder index]]

## Standard sections

```text
1. Cover Page
   - Client name, engagement dates, report date
   - Classification level (Confidential)
   - Report version

2. Executive Summary (1-2 pages)
   - Business impact summary for C-level audience
   - Overall risk rating
   - Key findings count by severity
   - Strategic recommendations (3-5 max)

3. Scope and Methodology
   - In-scope targets
   - Testing methodology (PTES, OWASP, etc.)
   - Tools used
   - Testing window / dates

4. Summary of Findings
   - Table: ID, Title, Severity, Status
   - Risk heat map or chart

5. Detailed Findings (bulk of the report)
   - One section per finding using [[08 - Finding Template|finding template]]

6. Appendices
   - Raw tool output (selected)
   - Screenshots
   - Remediation priority matrix
   - Glossary of terms
```

## See also

- [[07 - Executive Summary Writing]] — how to write the exec summary
- [[08 - Finding Template]] — per-finding format
- [[12 - Reporting Tools (SysReptor PlexTrac etc)]] — tools to build reports
