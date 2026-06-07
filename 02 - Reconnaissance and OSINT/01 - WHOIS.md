---
tags: [pentest, recon, osint, whois, dns, both]
tool: whois
phase: 1
---
# WHOIS

Query domain/IP registration data — registrant, name servers, creation/expiry dates.

[[02 - Reconnaissance and OSINT/00 - README|Folder index]]

## Basic usage

```bash
# Domain lookup
whois example.com

# IP lookup
whois 203.0.113.10

# Specific WHOIS server
whois -h whois.arin.net 203.0.113.10
```

## Key fields to extract

| Field | Intelligence value |
| --- | --- |
| Registrant name / org | Who owns it — sometimes reveals parent companies |
| Registrant email | Phishing target, password reset pivot |
| Name servers | Hosting provider, potential zone transfer targets |
| Creation date | New domain = likely phishing or staging |
| Expiry date | Expiring soon = potential domain takeover |
| Registrar | Some registrars have weaker protections |

## WHOIS privacy bypass

Many domains use privacy services (e.g., "Domains By Proxy"). Workarounds:

- **Historical WHOIS** — check `whoishistory.com` or SecurityTrails for pre-privacy records
- **Reverse WHOIS** — search by registrant email/name across all domains
- **Certificate Transparency** — certs may list the real org name
- **SOA record** — `dig SOA example.com` sometimes leaks admin email

## Bulk / reverse lookups

```bash
# Reverse WHOIS by email (use web tools):
# - ViewDNS.info reverse WHOIS
# - SecurityTrails
# - DomainTools (paid, best coverage)

# Reverse by name server
whois -h whois.arin.net -- "n + NS1.EXAMPLE.COM"
```

## Regional Internet Registries

| Registry | Region | WHOIS server |
| --- | --- | --- |
| ARIN | North America | whois.arin.net |
| RIPE | Europe/Middle East | whois.ripe.net |
| APNIC | Asia-Pacific | whois.apnic.net |
| LACNIC | Latin America | whois.lacnic.net |
| AFRINIC | Africa | whois.afrinic.net |

## See also

- [[02 - DNS Recon Basics]] — dig deeper into DNS after WHOIS
- [[03 - Certificate Transparency]] — another ownership discovery vector
