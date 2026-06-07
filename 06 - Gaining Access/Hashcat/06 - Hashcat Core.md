---
tags: [pentest, hashcat, password-cracking, hashes, initial-access]
tool: hashcat
phase: 5
---
# Hashcat - Core

GPU-accelerated password recovery. The standard offline cracker. Full mode-ID and attack-mode tables live in [[20 - Hashcat Mode IDs|Mode IDs]] and [[21 - Hashcat Attack Modes|Attack Modes]].

[[06 - Gaining Access/00 - README|Folder index]]

First you need to discover the hash type
Alternatively, [hashID](https://github.com/psypanda/hashID) can be used to quickly identify the hashcat hash type by specifying the `-m` argument.

need to save hash to hash file 
echo '1b0556a75770563578569ae21392630c' > /tmp/hash.txt

Then make the hashcat command.
hashcat -m 0 -a 0 /tmp/hash.txt /usr/share/wordlists/rockyou.txt -O

gunzip to unzip .gz

EXAMPLE COMMAND USING HASHCAT RULESETS
hashcat -a 0 -m 0 1b0556a75770563578569ae21392630c /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule
## Install / verify

```bash
hashcat --version
hashcat -I                  # show OpenCL / CUDA devices
hashcat -b -m 1000          # benchmark NTLM
```

## Skeleton

```bash
hashcat -m <mode> -a <attack> <hashfile> <wordlist|mask> [opts]
```

| Flag | Meaning |
| --- | --- |
| `-m N` | hash mode (see [[20 - Hashcat Mode IDs]]) |
| `-a N` | attack mode (0=wordlist, 1=combinator, 3=mask, 6=hybrid w+m, 7=hybrid m+w) |
| `-w 1..4` | workload (4 = max GPU, will heat) |
| `--status` | live progress |
| `--show` | show cracked results |
| `--left` | show un-cracked hashes |
| `--restore` | resume from session |
| `--session NAME` | name the session |
| `-o cracked.txt` | output |
| `--username` | for hash files in `user:hash` format |
| `-O` | optimized kernel (faster, but limits password length to ~31) |
| `--force` | run even if hashcat thinks the platform isn't supported (CPU mode) |

## Five attacks you'll run constantly

### 1. Wordlist (most common)

```bash
hashcat -m 0 hash.txt /usr/share/wordlists/rockyou.txt
hashcat -m 1000 ntlm.txt /usr/share/wordlists/rockyou.txt
```

### 2. Wordlist + rules

```bash
hashcat -m 1000 ntlm.txt rockyou.txt -r /usr/share/hashcat/rules/best64.rule
hashcat -m 1000 ntlm.txt rockyou.txt -r /usr/share/hashcat/rules/d3ad0ne.rule
hashcat -m 1000 ntlm.txt rockyou.txt -r OneRuleToRuleThemAll.rule
```

Rules transform wordlist entries: `password` → `Password1!`, `P@ssword`, `passwordXYZ`, etc.

### 3. Mask brute (`-a 3`)

```bash
# 8 chars, lower + digit
hashcat -a 3 -m 0 hash.txt -1 ?l?d ?1?1?1?1?1?1?1?1

# Increment 6..10 chars all-printable
hashcat -a 3 -m 0 hash.txt ?a?a?a?a?a?a --increment --increment-min 6 --increment-max 10
```

Mask charsets:

| Token | Charset |
| --- | --- |
| `?l` | a-z |
| `?u` | A-Z |
| `?d` | 0-9 |
| `?s` | symbols |
| `?a` | ?l?u?d?s |
| `?b` | any byte |

Custom charset: `-1 ?l?d`, then use `?1` in the mask.

### 4. Hybrid `-a 6` (wordlist + mask appended)

```bash
# rockyou + 2 digits
hashcat -a 6 -m 0 hash.txt rockyou.txt ?d?d
# Examples: password00, password01, ..., password99
```

### 5. Hybrid `-a 7` (mask + wordlist)

```bash
hashcat -a 7 -m 0 hash.txt ?d?d rockyou.txt
# Examples: 00password, 01password, ...
```

## Speed / heat management

```bash
# Workload profiles
-w 1   # low / desktop use
-w 2   # default
-w 3   # high (recommended for dedicated cracking)
-w 4   # max (overheats laptops; use cooling)

# Limit GPU temperature
hashcat ... --hwmon-temp-abort 95

# Use specific devices
hashcat ... -d 1,2          # use device 1 and 2 only

# Optimized kernel (faster, password-length-limited)
hashcat ... -O
```

## Session management

```bash
hashcat -m 1000 hash.txt rockyou.txt --session=corp1
# Ctrl+C, OS reboot, whatever
hashcat --session=corp1 --restore
```

## After a crack

```bash
hashcat -m 1000 ntlm.txt --show              # list cracked
hashcat -m 1000 ntlm.txt --show -o pots.txt  # save to file
hashcat -m 1000 ntlm.txt --left              # remaining un-cracked

# The potfile lives at ~/.local/share/hashcat/hashcat.potfile
cat ~/.local/share/hashcat/hashcat.potfile
```

## Output formatting

```bash
--outfile-format 2     # password only
--outfile-format 1,2   # hash:plaintext
--username             # for files like  user:hash  →  user:hash:plaintext

# Show only Domain Users (skip machine accounts ending in $)
hashcat -m 1000 ntds.hash --show --username | grep -v '\$:'
```

## When you see X, do Y

| Symptom | Action |
| --- | --- |
| `Salt-length exception` | Wrong `-m` mode. Check hash with `hashid`. |
| `Hash 'XYZ': Token length exception` | Hash file has extra column - try `--username`, or strip user prefix. |
| All hashes return un-cracked after rockyou | Try rules (`best64`, `OneRule`). Then targeted custom wordlist (company name + years). |
| GPU not used (CPU only running) | Driver / OpenCL issue. Run `hashcat -I` to see devices. |
| `* Device #1: This is an unstable plugin!` warning + `--force` needed | OK in lab; means you're using CPU-only or non-standard GPU. |
| Speed is much slower than benchmark | Other GPU work running, or many salts (NTLMv2 has per-message salt → ~1/N speed for N hashes). |

## See also

- [[07 - John the Ripper Core]] - alternative cracker, better for some formats
- [[20 - Hashcat Mode IDs]] - which `-m` to use
- [[22 - Hashcat Masks]]
- [[23 - Hashcat Rules]]
