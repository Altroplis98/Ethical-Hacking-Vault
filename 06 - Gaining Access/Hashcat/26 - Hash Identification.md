---
tags: [pentest, cracking, hash-id, identification, both, initial-access]
type: cheatsheet
phase: 7
---
# Hash Identification

Figure out what type of hash you're looking at before cracking.

[[11 - Shells Transfer Hashes/00 - README|Folder index]]

## Tools

### hashid

```bash
hashid '<hash_string>'
hashid -m '<hash_string>'    # show hashcat mode number
```

### hash-identifier

```bash
hash-identifier
# paste the hash when prompted
```

### hashcat example hashes

```bash
hashcat -m <mode> --example-hashes
# compare your hash format to the example
```

### Name-That-Hash (nth)

```bash
pip install name-that-hash
nth -t '<hash_string>'       # identifies + suggests hashcat/john modes
```

## Visual identification

| Pattern | Likely hash |
| --- | --- |
| 32 hex chars | MD5 |
| 40 hex chars | SHA-1 |
| 64 hex chars | SHA-256 |
| 128 hex chars | SHA-512 |
| `$1$salt$hash` | md5crypt |
| `$5$salt$hash` | sha256crypt |
| `$6$salt$hash` | sha512crypt (Linux shadow) |
| `$2a$` or `$2b$` | bcrypt |
| 32 hex chars (no salt context) | NTLM or MD5 |
| `$krb5tgs$23$*...` | Kerberoast |
| `$krb5asrep$23$...` | AS-REP Roast |
| `user::domain:challenge:response` | NetNTLMv2 |
| Starts with `WPA*` | WPA handshake/PMKID |

## NTLM vs MD5

Both are 32 hex characters. Context determines which:

- Dumped from SAM / NTDS.dit / secretsdump → NTLM (`-m 1000`)
- Dumped from a web app database → likely MD5 (`-m 0`)

## See also

- [[20 - Hashcat Mode IDs]]
- [[24 - John the Ripper Reference]]
