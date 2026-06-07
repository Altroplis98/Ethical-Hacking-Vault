---
tags: [pentest, cracking, hashcat, attack-modes, both, initial-access]
tool: hashcat
phase: 7
---
# Hashcat Attack Modes

The `-a` flag selects the attack strategy. Each mode trades speed vs coverage.

[[11 - Shells Transfer Hashes/00 - README|Folder index]]

## Attack mode summary

| `-a` | Name | Description |
| --- | --- | --- |
| 0 | Dictionary | Try each word in a wordlist |
| 1 | Combinator | Combine two wordlists (word1+word2) |
| 3 | Brute-force / Mask | Try all combinations matching a pattern |
| 6 | Hybrid wordlist+mask | Wordlist word + appended mask |
| 7 | Hybrid mask+wordlist | Prepended mask + wordlist word |
| 9 | Association | One candidate per hash (targeted) |

## Dictionary (`-a 0`)

```bash
hashcat -m 1000 hashes.txt /usr/share/wordlists/rockyou.txt

# With rules (multiplies effectiveness)
hashcat -m 1000 hashes.txt rockyou.txt -r best64.rule
```

## Combinator (`-a 1`)

```bash
# Tries every combination of word1+word2
hashcat -m 0 hashes.txt wordlist1.txt wordlist2.txt
# e.g., "summer" + "2024" = "summer2024"
```

## Brute-force / Mask (`-a 3`)

```bash
# 8-character lowercase
hashcat -m 0 hashes.txt -a 3 '?l?l?l?l?l?l?l?l'

# 4-digit PIN
hashcat -m 0 hashes.txt -a 3 '?d?d?d?d'

# Common pattern: Word + 2 digits + symbol
hashcat -m 0 hashes.txt -a 3 '?u?l?l?l?l?d?d?s'
```

## Hybrid (`-a 6` and `-a 7`)

```bash
# Wordlist + 4 digits appended
hashcat -m 0 hashes.txt -a 6 rockyou.txt '?d?d?d?d'
# "password" → "password0000" through "password9999"

# 2 digits prepended + wordlist
hashcat -m 0 hashes.txt -a 7 '?d?d' rockyou.txt
# "00password" through "99password"
```

## Recommended attack order

```text
1. Dictionary (rockyou.txt)
2. Dictionary + rules (best64, dive, OneRuleToRuleThemAll)
3. Hybrid wordlist + common suffixes (?d?d, ?d?d?d?d, !?d?d)
4. Mask attack (if you know the password policy)
5. Combinator (two short wordlists)
6. Pure brute-force (last resort — slow for 8+ chars)
```

## See also

- [[22 - Hashcat Masks]]
- [[23 - Hashcat Rules]]
- [[20 - Hashcat Mode IDs]]
