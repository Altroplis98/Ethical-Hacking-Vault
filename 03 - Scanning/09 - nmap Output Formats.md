---
tags: [pentest, scanning, nmap, output, reporting, both, recon]
tool: nmap
phase: 2
---
# nmap Output Formats

Save scan results properly. Different formats serve different purposes.

[[03 - Scanning/00 - README|Folder index]]

## Output flags

```bash
nmap -oN scan.nmap 10.10.10.10     # Normal (human-readable)
nmap -oG scan.gnmap 10.10.10.10    # Grepable (pipe-friendly)
nmap -oX scan.xml 10.10.10.10      # XML (tool-friendly)
nmap -oA scan_base 10.10.10.10     # All three at once
nmap -oS scan.skid 10.10.10.10     # Script kiddie (joke format)
```

## When to use each format

| Format | Use case |
| --- | --- |
| `-oN` | Reading results, including in reports |
| `-oG` | Grepping for open ports, piping to other tools |
| `-oX` | Importing into Metasploit, OpenVAS, report generators |
| `-oA` | Always — costs nothing extra, gives you all options |

## Grepable output recipes

```bash
# Extract all open ports for a host
grep -oP '\d+/open' scan.gnmap | cut -d/ -f1 | tr '\n' ','

# Find all hosts with port 445 open
grep '445/open' scan.gnmap | awk '{print $2}'

# Count open ports per host
grep 'Ports:' scan.gnmap | awk '{print $2, gsub(/open/,"&")}' 
```

## XML to other formats

```bash
# XML to HTML report
xsltproc scan.xml -o report.html

# Import into Metasploit
msf> db_import scan.xml
```

## See also

- [[05 - nmap Basics]] — scan commands that produce this output
