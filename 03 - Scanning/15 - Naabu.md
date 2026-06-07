---
tags: [pentest, scanning, naabu, ports, both, recon]
tool: naabu
phase: 2
---
# Naabu

Fast port scanner by ProjectDiscovery. Designed to integrate with their other tools (subfinder, httpx, nuclei).

[[03 - Scanning/00 - README|Folder index]]

## Install

```bash
go install -v github.com/projectdiscovery/naabu/v2/cmd/naabu@latest
```

## Usage

```bash
# Scan a host
naabu -host 10.10.10.10

# Top ports
naabu -host 10.10.10.10 -top-ports 100

# All ports
naabu -host 10.10.10.10 -p -

# Specific ports
naabu -host 10.10.10.10 -p 80,443,8080

# From stdin (pipeline)
echo "10.10.10.10" | naabu -silent

# From subfinder
subfinder -d example.com -silent | naabu -silent

# Output
naabu -host 10.10.10.10 -o ports.txt
naabu -host 10.10.10.10 -json -o ports.json
```

## Key flags

| Flag | Purpose |
| --- | --- |
| `-host target` | Target host |
| `-list file` | File of targets |
| `-p ports` | Port specification (- for all) |
| `-top-ports N` | Top N ports |
| `-silent` | Clean output |
| `-o file` | Output file |
| `-json` | JSON output |
| `-rate N` | Rate limit |
| `-nmap-cli` | Run nmap on discovered ports |

## Pipeline integration

```bash
# Full discovery pipeline
subfinder -d example.com -silent | \
  naabu -silent -top-ports 1000 | \
  httpx -silent -title -status-code | \
  nuclei -t cves/
```

## See also

- [[13 - masscan]] — faster for raw port scanning
- [[14 - RustScan]] — nmap integration focus
