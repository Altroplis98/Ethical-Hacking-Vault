---
tags: [pentest, cracking, hashcat, masks, brute-force, both, initial-access]
tool: hashcat
phase: 7
---
# Hashcat Masks

Define character patterns for brute-force and hybrid attacks. Masks describe what each character position can be.

[[11 - Shells Transfer Hashes/00 - README|Folder index]]

## Built-in charsets

| Placeholder | Characters |
| --- | --- |
| `?l` | abcdefghijklmnopqrstuvwxyz |
| `?u` | ABCDEFGHIJKLMNOPQRSTUVWXYZ |
| `?d` | 0123456789 |
| `?s` | ` ~!@#$%^&*()-_=+[{]}\|;:'",<.>/?` + space |
| `?a` | `?l?u?d?s` (all printable ASCII) |
| `?h` | 0123456789abcdef |
| `?H` | 0123456789ABCDEF |
| `?b` | 0x00–0xff (all bytes) |

## Custom charsets (`-1`, `-2`, `-3`, `-4`)

```bash
# -1 = uppercase + digits
hashcat -m 0 hashes.txt -a 3 -1 '?u?d' '?1?1?1?1?1?1'

# -1 = specific characters
hashcat -m 0 hashes.txt -a 3 -1 'aeiou' '?1?1?1?1'
```

## Common mask patterns

| Pattern | Mask | Example |
| --- | --- | --- |
| 8-digit PIN | `?d?d?d?d?d?d?d?d` | 12345678 |
| Ulll0000 (Word+digits) | `?u?l?l?l?d?d?d?d` | Pass1234 |
| Season+year | Custom wordlist better | Summer2024 |
| Name+punctuation+digits | `?u?l?l?l?l?s?d?d` | Admin!23 |

## Increment mode

```bash
# Try masks from 1 to 8 characters
hashcat -m 0 hashes.txt -a 3 --increment --increment-min 4 --increment-max 8 '?a?a?a?a?a?a?a?a'
```

## Mask files (.hcmask)

```text
# masks.hcmask — one mask per line
?d?d?d?d
?d?d?d?d?d?d
?u?l?l?l?l?l?d?d
?u?l?l?l?l?l?d?d?s
```

```bash
hashcat -m 0 hashes.txt -a 3 masks.hcmask
```

## See also

- [[21 - Hashcat Attack Modes]]
- [[23 - Hashcat Rules]]
