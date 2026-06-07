---
tags: [pentest, enumeration, ad, bloodhound, graph, active-directory, recon, windows]
tool: bloodhound
phase: 3
---
# BloodHound

Graph-based Active Directory attack path analysis. Visualizes relationships between users, groups, computers, and permissions to find paths to Domain Admin.

[[04 - Enumeration/00 - README|Folder index]]

## Install

```bash
# BloodHound CE (Community Edition) — Docker
curl -L https://ghst.ly/getbhce | docker compose -f - up

# Legacy BloodHound
sudo apt install bloodhound -y
sudo neo4j start
# Default neo4j creds: neo4j / neo4j (change on first login)
bloodhound
```

## Workflow

1. **Collect** data with SharpHound or bloodhound-python (see [[10 - SharpHound]], [[11 - bloodhound-python]])
2. **Import** the collected JSON/ZIP into BloodHound
3. **Analyze** using built-in queries and custom Cypher

## Key built-in queries

| Query | What it finds |
| --- | --- |
| Find all Domain Admins | Members of DA group |
| Shortest Paths to Domain Admins | Attack paths from owned users to DA |
| Shortest Paths to High Value Targets | Paths to anything marked high value |
| Find Kerberoastable Users | Users with SPNs |
| Find AS-REP Roastable Users | Users without pre-auth |
| Find Computers with Unconstrained Delegation | Delegation abuse targets |
| Shortest Paths from Owned Principals | What can I reach from what I have? |

## Custom Cypher queries

```cypher
// Users with DCSync rights
MATCH (n)-[:MemberOf|HasSIDHistory*1..]->(m:Group)
WHERE m.objectid ENDS WITH '-512' OR m.objectid ENDS WITH '-519'
RETURN n.name

// All Kerberoastable users with path to DA
MATCH (u:User {hasspn:true}), (g:Group {name:"DOMAIN ADMINS@CORP.LOCAL"}),
p=shortestPath((u)-[*1..]->(g))
RETURN p

// Find computers where a specific user has admin rights
MATCH (u:User {name:"USER@CORP.LOCAL"})-[r:AdminTo]->(c:Computer) RETURN c.name
```

## Mark owned principals

Right-click a node → "Mark as Owned" → then query "Shortest Paths from Owned Principals" to find your next move.

## See also

- [[10 - SharpHound]] — Windows collector
- [[11 - bloodhound-python]] — Linux/remote collector
- [[07 - ldapsearch]] — manual LDAP queries for validation
