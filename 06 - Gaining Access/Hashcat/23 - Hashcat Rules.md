---
tags: [pentest, cracking, hashcat, rules, mutations, both, initial-access]
tool: hashcat
phase: 7
---
# Hashcat Rules

Rules mutate wordlist entries on the fly — capitalize, append digits, leet-speak, reverse, etc. Massively expands your effective wordlist without disk space.

[[11 - Shells Transfer Hashes/00 - README|Folder index]]

## Using rules

```bash
hashcat -m 1000 hashes.txt rockyou.txt -r /usr/share/hashcat/rules/best64.rule
```

## Best rule files (included with hashcat)

| File | Rules | Description |
| --- | --- | --- |
| `best64.rule` | 64 | Best all-around — try first |
| `d3ad0ne.rule` | 34,000+ | Comprehensive |
| `dive.rule` | 99,000+ | Very thorough, slow |
| `toggles1-5.rule` | ~30 | Toggle case patterns |
| `leetspeak.rule` | ~20 | a→@, e→3, etc. |

## Community rules

| File | Description |
| --- | --- |
| `OneRuleToRuleThemAll.rule` | Community-curated, very effective |
| `pantagrule.rule` | Generated from real password leaks |

## Common rule functions

| Function | Meaning | Example |
| --- | --- | --- |
| `:` | Do nothing (passthrough) | password → password |
| `l` | Lowercase all | PASSWORD → password |
| `u` | Uppercase all | password → PASSWORD |
| `c` | Capitalize first | password → Password |
| `t` | Toggle case | password → PASSWORD (alt each) |
| `$X` | Append char X | password → password1 |
| `^X` | Prepend char X | password → 1password |
| `r` | Reverse | password → drowssap |
| `d` | Duplicate | password → passwordpassword |
| `sa@` | Replace a with @ | password → p@ssword |
| `se3` | Replace e with 3 | leet → l33t |

## Write custom rules

```text
# my.rule
:
c
c$1
c$1$2$3
c$!
sa@se3si1so0
```

```bash
hashcat -m 1000 hashes.txt rockyou.txt -r my.rule
```

## Stack multiple rule files

```bash
hashcat -m 1000 hashes.txt rockyou.txt -r rule1.rule -r rule2.rule
# Applies rule1 THEN rule2 — multiplicative: 64 × 64 = 4096 mutations per word
```

> [!warning] Rule stacking is multiplicative
> Two 64-rule files = 4096 mutations per word. Three = 262,144. This can make cracking very slow. Stack carefully.

## See also

- [[21 - Hashcat Attack Modes]]
- [[22 - Hashcat Masks]]
- [[24 - John the Ripper Reference]]
