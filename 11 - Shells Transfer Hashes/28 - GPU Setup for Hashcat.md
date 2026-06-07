---
tags: [pentest, cracking, hashcat, gpu, setup, both, initial-access]
tool: hashcat
phase: 7
---
# GPU Setup for Hashcat

Configure GPU drivers and hashcat for maximum cracking performance.

[[00 - README|Folder index]]

## NVIDIA setup (recommended)

```bash
# Install NVIDIA driver
sudo apt install nvidia-driver

# Verify GPU is detected
nvidia-smi

# CUDA toolkit (optional — hashcat uses OpenCL by default)
sudo apt install nvidia-cuda-toolkit

# Verify hashcat sees the GPU
hashcat -I
```

## AMD setup

```bash
# Install ROCm
sudo apt install rocm-opencl-runtime

# Verify
hashcat -I
```

## Benchmarking

```bash
# Benchmark all modes
hashcat -b

# Benchmark specific mode
hashcat -b -m 1000    # NTLM speed
hashcat -b -m 22000   # WPA speed
hashcat -b -m 1800    # sha512crypt speed
```

## Typical speeds (RTX 4090)

| Mode | Hash type | Speed |
| --- | --- | --- |
| 0 | MD5 | ~160 GH/s |
| 1000 | NTLM | ~280 GH/s |
| 1800 | sha512crypt | ~2.5 MH/s |
| 3200 | bcrypt | ~180 kH/s |
| 22000 | WPA | ~1.6 MH/s |
| 13100 | Kerberoast | ~2.5 GH/s |

## Performance tuning

| Flag | Effect |
| --- | --- |
| `-w 3` | High workload (GPU dedicated to cracking) |
| `-w 4` | Nightmare mode (may freeze desktop) |
| `-O` | Optimized kernels (faster but max password length = 32) |
| `-d 1` | Select specific GPU device |
| `--force` | Ignore warnings (use cautiously) |

## Cloud GPU cracking

When you don't have local GPU hardware:

- **AWS**: p3/p4 instances (V100/A100)
- **Google Cloud**: A100 GPU instances
- **Vast.ai**: Cheap GPU rental
- **Penguin**: Pre-configured hashcat cloud

```bash
# Example: hashcat on a cloud GPU
hashcat -m 1000 hashes.txt rockyou.txt -w 3 -O
```

> [!tip] Cost vs time
> An RTX 4090 costs ~$1600. Renting GPU time costs ~$1-3/hour. If you crack hashes regularly, buying is cheaper within a year.

## See also

- [[20 - Hashcat Mode IDs]]
- [[21 - Hashcat Attack Modes]]
