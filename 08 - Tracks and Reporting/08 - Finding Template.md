---
tags: [pentest, reporting, finding, template]
phase: 7
---

# Finding Template

Standard format for documenting each vulnerability in the pentest report.

[[08 - Tracks and Reporting/00 - README|Folder index]]

## Template

```markdown
### [ID] - [Finding Title]

**Severity:** Critical / High / Medium / Low / Informational
**CVSS Score:** X.X (v3.1)
**CVSS Vector:** AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H
**Status:** Open / Remediated / Accepted Risk
**Affected Systems:** [list of IPs/hosts/URLs]

#### Description
[2-3 sentences explaining the vulnerability in business terms]

#### Technical Details
[Step-by-step reproduction with commands and screenshots]

#### Evidence
[Screenshots with timestamps, command output]

#### Impact
[What an attacker could do with this vulnerability]

#### Remediation
[Specific, actionable fix — not just "patch the system"]

#### References
- [CVE link]
- [Vendor advisory]
- [CWE link]
```

## Example finding

```markdown
### F-001 - Unauthenticated Remote Code Execution via Apache Path Traversal

**Severity:** Critical
**CVSS Score:** 9.8
**Status:** Open
**Affected Systems:** 10.10.10.15 (web01.corp.local)

#### Description
The Apache HTTP Server version 2.4.49 running on web01 is vulnerable to
a path traversal attack that allows unauthenticated remote code execution.

#### Technical Details
[commands, curl requests, screenshots]

#### Impact
An external attacker could execute arbitrary commands as the www-data user,
potentially pivoting to internal network resources.

#### Remediation
Upgrade Apache HTTP Server to version 2.4.51 or later.
```

## See also

- [[09 - CVSS 3.1 and 4.0 Scoring]] — how to calculate CVSS scores
- [[10 - Attack Narrative]] — telling the story of the attack chain
