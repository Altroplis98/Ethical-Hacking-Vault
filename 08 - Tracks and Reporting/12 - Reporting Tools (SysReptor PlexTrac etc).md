---
tags: [pentest, reporting, tools, sysreptor, plextrac]
phase: 7
---

# Reporting Tools

Tools that help generate professional pentest reports from structured findings data.

[[08 - Tracks and Reporting/00 - README|Folder index]]

## Free / open source

| Tool | Description |
| --- | --- |
| **SysReptor** | Open-source pentest reporting. Template-based, supports collaboration, exports to PDF. Self-hosted. |
| **Ghostwriter** | SpecterOps reporting + project management. Tracks findings, generates reports, manages infrastructure. |
| **Pwndoc** | Markdown-based pentest report generator. Simple, Docker-deployable. |
| **Dradis** | Community edition available. Imports from tools (nmap, Burp, Nessus), builds reports. |

## Commercial

| Tool | Description |
| --- | --- |
| **PlexTrac** | SaaS platform for pentest management and reporting. Integrates with scanners. |
| **AttackForge** | Pentest management platform. Collaboration, templates, analytics. |
| **Cobalt Strike (reporting)** | Built-in reporting for red team engagements. |

## DIY approach

For smaller shops or independent testers:
- Write findings in Markdown → convert with Pandoc to DOCX/PDF
- Use Obsidian (this vault!) as the note-taking system
- Template in Word/Google Docs for final formatting

```bash
# Markdown to Word
pandoc report.md -o report.docx --reference-doc=template.docx

# Markdown to PDF
pandoc report.md -o report.pdf --pdf-engine=xelatex
```

## See also

- [[06 - Report Structure]] — what the report should contain
