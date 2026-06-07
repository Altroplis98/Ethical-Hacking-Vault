---
tags: [pentest, wireless, hashcat, wpa, cracking, gpu, both, initial-access]
tool: hashcat
phase: 5
---
# Hashcat WPA Mode 22000

GPU-accelerated cracking of WPA/WPA2 handshakes and PMKIDs using hashcat mode 22000 (the unified WPA format that replaced modes 2500 and 16800).

[[10 - Wireless/00 - README|Folder index]]

## Install / verify

```bash
hashcat --version
# Needs GPU drivers (NVIDIA: nvidia-driver + CUDA; AMD: ROCm)
hashcat -I    # list detected devices
```

## Convert capture to hash format

```bash
# From .cap / .pcap (aircrack capture)
hcxpcapngtool capture-01.cap -o hash.22000

# From .pcapng (hcxdumptool capture)
hcxpcapngtool capture.pcapng -o hash.22000
```

Install `hcxtools` if missing:

```bash
sudo apt install hcxtools
```

## Crack

```bash
# Dictionary
hashcat -m 22000 hash.22000 /usr/share/wordlists/rockyou.txt

# Dictionary + rules
hashcat -m 22000 hash.22000 rockyou.txt -r /usr/share/hashcat/rules/best64.rule

# Brute-force 8-digit numeric (common for ISP default passwords)
hashcat -m 22000 hash.22000 -a 3 '?d?d?d?d?d?d?d?d'

# Brute-force 8-char lowercase + digits
hashcat -m 22000 hash.22000 -a 3 '?h?h?h?h?h?h?h?h'
```

## Useful flags

| Flag | Meaning |
| --- | --- |
| `-m 22000` | WPA-PBKDF2-PMKID+EAPOL |
| `-a 0` | Dictionary attack |
| `-a 3` | Brute-force / mask attack |
| `-r rules.rule` | Apply rule file |
| `-w 3` | Workload profile (3 = high) |
| `--status` | Show status during cracking |
| `--potfile-path` | Custom potfile location |
| `-o cracked.txt` | Output cracked passwords |
| `--show` | Show already-cracked hashes |

## Mask charsets

| Placeholder | Charset |
| --- | --- |
| `?d` | Digits 0-9 |
| `?l` | Lowercase a-z |
| `?u` | Uppercase A-Z |
| `?s` | Special characters |
| `?a` | All printable |
| `?h` | Hex lowercase 0-9a-f |

## Performance tips

- Use `-w 3` or `-w 4` for maximum GPU utilization
- WPA cracking is slow (~500 kH/s on a good GPU) because of PBKDF2 with 4096 iterations
- Rules multiply your wordlist effectively without disk space
- For ISP routers, try known default password patterns first (8-digit numeric, hex patterns)

## See also

- [[06 - aircrack-ng]]
- [[07 - hcxdumptool PMKID]]
