---
tags: [pentest, recon, osint, maltego, both]
tool: maltego
phase: 1
---
# Maltego

Commercial graph-based OSINT and link analysis tool. Visualizes relationships between entities (people, domains, IPs, social accounts).

[[02 - Reconnaissance and OSINT/00 - README|Folder index]]

## Install / verify

```bash
# Pre-installed on Kali (Community Edition)
which maltego || sudo apt install maltego -y
```

## Editions

| Edition | Cost | Limits |
| --- | --- | --- |
| Community (CE) | Free | 12 results per transform, limited transforms |
| Pro | Paid | Full results, all transforms |
| Organization | Enterprise | Team features, custom transforms |

## Core concepts

| Term | Meaning |
| --- | --- |
| Entity | A data point — domain, IP, email, person, phone, etc. |
| Transform | An action that takes an entity and produces related entities |
| Graph | The visual network of entities and their relationships |
| Machine | A chain of transforms that runs automatically |

## Basic workflow

1. Create a new graph
2. Drag a Domain entity onto the canvas
3. Right-click → Run Transform → select category
4. Review results, expand interesting nodes
5. Repeat — build the relationship map

## Useful transforms

| Transform | Input → Output |
| --- | --- |
| DNS from Domain | Domain → DNS records |
| Emails from Domain | Domain → email addresses |
| To IP Address | Domain → resolved IPs |
| WHOIS | Domain/IP → registration data |
| To Person | Email → person names |
| Social Links | Person → social media profiles |

## When Maltego shines

- Visualizing complex relationships between entities
- Link analysis — who is connected to what
- Presenting OSINT findings to non-technical stakeholders
- Social engineering target mapping

## See also

- [[12 - Spiderfoot]] — automated OSINT without the graph visualization
- [[11 - Recon-ng]] — CLI-based, more scriptable
