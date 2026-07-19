---
tags: [pentest, recon, dns, enumeration, both]
phase: 1
---
# DNS Recon Basics

Foundational DNS queries that reveal infrastructure, mail servers, and potential zone transfer misconfigs.

[[02 - Reconnaissance and OSINT/00 - README|Folder index]]

## Workflow: starting from just an IP

You often have an IP with `53/tcp open` (BIND/Samba) but **no domain name yet** — and AXFR needs a zone name. Work the chain in order:

```bash
IP=10.129.203.6

# 1. Point dig AT the server (@) and ask it to reverse-resolve itself.
#    The PTR answer often leaks the domain suffix.
dig @$IP -x $IP

# 2. Fingerprint the server (see BIND version section below)
dig @$IP version.bind CHAOS TXT

# 3. Once you have a candidate zone (e.g. from PTR, SOA, or the box name),
#    ask the server who its name servers / SOA are:
dig @$IP <domain> SOA
dig @$IP <domain> NS

# 4. Now attempt the zone transfer against that server:
dig AXFR @$IP <domain>
```

> [!tip] The `@` is the whole trick
> `@$IP` means "ask THIS server directly" instead of your default resolver. Without it, `dig` queries your own DNS and never touches the target. Every command in an active DNS recon should use `@target`.

> [!info] Reading the reverse lookup
> Look in the `ANSWER SECTION` for a `PTR` record like `6.203.129.10.in-addr.arpa. → host.internal.domain`. The suffix (`internal.domain`) is your candidate zone for step 4. No answer ≠ dead end — the box name or an `SOA` query can still hand you the zone.

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

## Fingerprint the DNS server (BIND version)

```bash
# CHAOS-class query — BIND leaks its version string here by default
dig @$IP version.bind CHAOS TXT +short
dig @$IP version.bind txt chaos       # long form

# nmap equivalent
nmap -sSU -p53 --script dns-nsid $IP
```

> [!note] Why it matters
> The version string (e.g. `9.16.1-Ubuntu`) maps a BIND build to known CVEs and tells you the OS. Well-hardened servers hide it with `version "not disclosed";` in named.conf — absence is itself a finding.

## NSEC / NSEC3 zone walking (DNSSEC)

If AXFR is blocked but the zone is DNSSEC-signed, you can still enumerate records by "walking" the NSEC chain — each NSEC record points to the next name in the zone.

```bash
# Check for DNSSEC / NSEC first
dig @$IP <domain> DNSKEY +short
dig @$IP nonexistent.<domain> +dnssec   # NSEC record reveals neighbouring names

# Automated walkers
nsec3walker <domain>
ldns-walk @$IP <domain>
```

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

### subbrute (against an internal/target name server)

When AXFR is refused but the box **is** the authoritative name server (e.g. an HTB target running BIND), you can't reach it from public resolvers — you must force the brute-forcer to query that specific server.

```bash
# Setup
git clone https://github.com/TheRook/subbrute.git
cd subbrute

# CRITICAL: point the resolver file at the TARGET, not 8.8.8.8
# (public resolvers don't know about internal zones like *.htb)
echo "$IP" > ./resolvers.txt

# Run: -s = names wordlist, -r = resolver file
./subbrute.py inlanefreight.htb -s ./names.txt -r ./resolvers.txt
python3 ./subbrute.py inlanefreight.htb -s ./names.txt -r ./resolvers.txt   # if not executable

# Bigger wordlist
./subbrute.py inlanefreight.htb -s /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -r ./resolvers.txt

# Brute NAMES and pull a specific record type in one pass (great for flag TXT records)
./subbrute.py inlanefreight.htb -s ./names.txt -r ./resolvers.txt --type TXT
./subbrute.py inlanefreight.htb -s ./names.txt -r ./resolvers.txt -p        # -p = print data from found records
```

> [!important] You can't brute-force a record's *value*, only its *name*
> Brute-forcing enumerates the guessable namespace (subdomain labels). A TXT value is arbitrary free text — no wordlist guesses it. `--type TXT` does NOT guess TXT contents; it brute-forces subdomain **names** and prints whatever TXT each one happens to hold. Useful `--type` values: `CNAME, AAAA, TXT, SOA, MX`.

Every line printed back is a **live subdomain**. Then query each discovered name for its records (flags are usually TXT):

```bash
dig @$IP <found-sub>.inlanefreight.htb TXT
dig @$IP <found-sub>.inlanefreight.htb ANY
```

> [!note] Pure-dig equivalent (no tool install)
> Same idea, one line — brute names straight against the target server:
> ```bash
> while read sub; do
>   dig @$IP "$sub.inlanefreight.htb" +short | grep -v '^$' && echo "  -> $sub.inlanefreight.htb"
> done < /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
> ```

> [!warning] When to reach for subbrute
> Zone transfer (AXFR) failed **and** the apex `ANY`/`TXT` query returned no flag -> the record lives on an unlisted subdomain you have to guess. That's the subbrute trigger.

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
