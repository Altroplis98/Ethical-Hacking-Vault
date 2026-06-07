---
tags: [pentest, htb, methodology, moc, both]
type: workflow
---
# 12 - HTB / Lab Workflows

Methodology, pattern recognition, and walkthrough templates for HackTheBox / TryHackMe / OSCP-style boxes. **When stuck, open [[01 - When You See X Do Y]] first.**

[[00 - Vault Index|Home]]

## Files in this folder

### Core methodology
- [[01 - When You See X Do Y]] - the cheat-card. Pattern → next step.
- [[02 - General Methodology]] - the universal flow chart
- [[03 - Pre-Flight Checklist]] - before you `nmap`
- [[04 - Note-Taking Template]] - per-box notebook template
- [[05 - Flag Locations Reference]] - where flags live

### Box-type patterns
- [[06 - Linux Easy Pattern]]
- [[07 - Linux Hard Pattern]]
- [[08 - Windows Easy Pattern]]
- [[09 - Windows AD Pattern]]
- [[10 - Web-Heavy Box Pattern]]
- [[11 - Crypto Stego Box Pattern]]

### Anti-patterns
- [[12 - Common Pitfalls and Time Sinks]]
- [[13 - Rabbit Hole Detector]]

## The 30-second mental model

```text
EVERY box, at every stage, ask:
  1. What's running here that I haven't fully enumerated?
  2. What's the most likely intended path? (lowest version, default creds, public exploit)
  3. What artifact am I supposed to find here? (file, banner, hint, comment)
  4. If I were the box author, where would I hide the win?
```

> [!tip] If you've been on one finding for >30 minutes
> Open [[13 - Rabbit Hole Detector]]. Most "stuck" moments are because you skipped enumeration on a service, not because you missed an exploit.
