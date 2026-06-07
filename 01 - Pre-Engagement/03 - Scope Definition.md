---
tags: [pentest, pre-engagement, scope, legal]
phase: 0
---

# Scope Definition

The most important pre-engagement document. If a target isn't in scope, touching it is unauthorized access.

[[01 - Pre-Engagement/00 - README|Folder index]]

## In-scope vs out-of-scope

### In-scope examples

```text
External:
  - 203.0.113.0/24 (client's public range)
  - app.client.com, api.client.com, staging.client.com
  - AWS account 123456789012

Internal:
  - 10.10.0.0/16 (corporate LAN)
  - Active Directory domain: CORP.CLIENT.LOCAL
  - All Windows/Linux servers on VLAN 20

Web applications:
  - https://app.client.com (production, read-only testing)
  - https://staging.client.com (full testing including destructive)
```

### Out-of-scope examples

```text
  - 203.0.113.50 (shared hosting, third-party owned)
  - Any *.cloudflare.com infrastructure
  - Client's ISP or upstream provider
  - Employee personal devices
  - Physical security (unless separately scoped)
  - Third-party SaaS (Salesforce, O365) — unless authorization letter obtained
```

## Third-party authorization

If the client's infrastructure is hosted by a third party, you need authorization from BOTH:

| Scenario | Who authorizes |
| --- | --- |
| AWS-hosted app | Client + AWS (use AWS pen test policy — no pre-approval needed for most services as of 2024, but review current policy) |
| Azure-hosted app | Client + Microsoft (submit pen test notification form) |
| GCP-hosted app | Client + Google (review acceptable use policy) |
| Shared hosting | Client + hosting provider (written approval required) |
| CDN-fronted app | Client + CDN provider (Cloudflare, Akamai, etc.) |

> [!danger] Cloud provider policies change
> Always check the current cloud provider penetration testing policy before starting. AWS relaxed theirs significantly; Azure and GCP have their own forms.

## IP/domain verification

Before testing, verify ownership:

```bash
# WHOIS check
whois 203.0.113.10 | grep -i org

# Reverse DNS
dig -x 203.0.113.10

# ASN lookup
whois -h whois.radb.net -- '-i origin AS12345'

# Confirm the domain resolves to client IP
dig +short app.client.com
```

If anything resolves to an IP outside the documented scope, **stop and verify** with the client.

## Scope creep management

| Trigger | Response |
| --- | --- |
| You discover a new subdomain | Check if it resolves to in-scope IP. If yes, test. If no, report and ask. |
| Client asks "can you also test X?" mid-engagement | Formal scope change: update SOW, get sign-off, adjust timeline/cost |
| You pivot to a system not in original scope | Stop. Document the pivot path. Get authorization before continuing. |

## See also

- [[01 - NDA and SOW]] — where scope lives contractually
- [[02 - Rules of Engagement]] — what you can do within scope
- [[04 - Cloud Testing Policies]] — cloud-specific authorization details
