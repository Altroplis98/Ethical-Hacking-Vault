---
tags: [pentest, htb, methodology, antipattern, both]
type: workflow
---
# Rabbit Hole Detector

[[00 - README|Folder index]]

A short self-check to run when you suspect you've been chasing the wrong thing for too long.

## The 7 questions

Answer each honestly. If you fail 3+, you're in a rabbit hole.

1. **"Have I tested this technique for 60+ minutes with no measurable progress?"**
   - If yes → suspect rabbit hole.
2. **"Does this attack require a piece of information I don't have yet?"**
   - E.g., you're trying to crack a hash you haven't extracted, or relay against a host you can't reach.
3. **"Is the difficulty of this exploit higher than the box's rating?"**
   - Easy box + kernel exploit chain = wrong path.
4. **"Did I skip any port, banner, or page source during enumeration?"**
   - Be honest. Most rabbit holes start with skipped enumeration.
5. **"Have I read the actual error message on every failed attempt?"**
   - If you've been ignoring errors, you're guessing.
6. **"Am I assuming something that isn't proven?"**
   - "The web app must be the way in." Maybe FTP is.
7. **"Could a fresh `nmap -A -p- -Pn` reveal something I missed?"**
   - If you haven't done that, do it now.

## Recovery procedure

When you confirm you're in a rabbit hole:

```text
1. Save current state.
   - Write down: "Tried X via Y, failed because Z."
   - Bookmark any PoC / blog you found - it MAY come back.

2. Walk away physically (5-10 minutes).
   - Stand up, get water, no screens.
   - This isn't optional. Tunnel vision kills hours.

3. Re-baseline.
   - Re-read your nmap output line by line.
   - Re-read your enumeration notes for every service.

4. Re-enumerate from the top.
   - You'll find something you missed. Almost always.

5. Pick the most boring-looking finding.
   - Easy box solutions are almost never "exotic."
```

## Anti-tunnel-vision checklist

When opening a new tab / shell on a box, ask:

- Have I tried this exploit type on *any other open port*?
- Have I tried this credential on *any other service*?
- Is there a service I dismissed because "it doesn't look interesting"?
- Have I checked for HTTP virtual hosts? (Hostname header swap.)
- Have I checked for UDP services? (`-sU --top-ports 100`)
- Have I read the HTML / JS source code, not just rendered?

## "Boring is good" rule

The intended path almost always looks BORING in retrospect:

- Default creds on `/admin`
- An obviously-named backup file
- A SUID binary listed in GTFOBins
- A user in the `docker` group
- A web form that doesn't validate input

If your attack plan reads like a 30-page paper, you've drifted. The author wanted you to find something simple, not invent something clever.

## When to look at HTB official forum / Discord

- After exhausting your own enumeration AT LEAST twice.
- Look for *nudges*, not spoilers - search for "stuck on initial foothold," not "how to root [boxname]."
- Reading other people's hints often reveals which port/service you under-enumerated.

> [!tip] Save your "next time" notes
> Every rabbit hole costs hours, but it also teaches you a recognition pattern. Add to [[01 - When You See X Do Y]] every time you exit one - "When I see X, NEXT time I'll do Y first."
