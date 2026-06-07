---
tags: [pentest, recon, dns, enumeration, both]
phase: 1
---
# DNS Recon Basics

Foundational DNS queries that reveal infrastructure, mail servers, and potential zone transfer misconfigs.

[[02 - Reconnaissance and OSINT/00 - README|Folder index]]

## Essential record types

| Type | Purpose | Command |
| --- | --- | --- |
| A | IPv4 address | `dig A example.com` |
| AAAA | IPv6 address | `dig AAAA example.com` |
| MX | Mail servers | `dig MX example.com` |
| NS | Name servers | `dig NS example.com` |
| TXT | SPF, DKIM, DMARC, verification tokens | `dig TXT example.com` |
| SOA | Start of Authority (admin email, serial) | `dig SOA example.com` |
| CNAME | Aliases (subdomain takeover candidates) | `dig CNAME sub.example.com` |
| SRV | Service records (Kerberos, SIP, XMPP) | `dig SRV _kerberos._tcp.example.com` |
| PTR | Reverse lookup | `dig -x 203.0.113.10` |

## Zone transfer (AXFR)

The holy grail of DNS recon — if misconfigured, dumps the entire zone.

```bash
# Try zone transfer against each name server
dig NS example.com +short
dig AXFR example.com @ns1.example.com

# Automated attempt
dig AXFR @ns1.example.com example.com

# With host command
host -t axfr example.com ns1.example.com
```

> [!tip] Zone transfers are almost always blocked on external DNS
> But internal DNS servers often allow AXFR from any internal IP. Always try it on internal engagements.

## Reverse DNS sweep

```bash
# Sweep a /24 for PTR records
for i in $(seq 1 254); do
  dig -x 203.0.113.$i +short
done

# Or use dnsrecon
dnsrecon -r 203.0.113.0/24 -t rvl
```

## DNS brute-forcing

```bash
# Using dig + wordlist
while read sub; do
  dig +short "$sub.example.com" | grep -v "^$" && echo "  → $sub.example.com"
done < /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```

> [!tip] Prefer passive subdomain tools first
> Subfinder, Amass passive, and CT logs are stealthier. Only brute-force if scope allows active DNS queries.

## Useful dig flags

```bash
dig +short example.com          # Clean output, just the answer
dig +noall +answer example.com  # Answer section only
dig +trace example.com          # Full delegation chain
dig @8.8.8.8 example.com       # Query specific resolver
dig +dnssec example.com         # Check DNSSEC status
```

## See also

- [[01 - WHOIS]] — ownership data to pair with DNS
- [[04 - Subfinder]] — passive subdomain enumeration
- [[09 - dnsenum dnsrecon fierce]] — automated DNS enum tools
