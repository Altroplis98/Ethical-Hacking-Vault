---
tags: [pentest, enumeration, mongodb, database, recon]
tool: mongosh
phase: 3
---
# MongoDB Enumeration

MongoDB (port 27017). Often runs without authentication, exposing full database access.

[[04 - Enumeration/00 - README|Folder index]]

## Connect

```bash
# No auth (if exposed)
mongosh --host 10.10.10.10

# With auth
mongosh --host 10.10.10.10 -u admin -p 'password' --authenticationDatabase admin

# Legacy client
mongo --host 10.10.10.10
```

## Key commands

```javascript
// List databases
show dbs

// Use a database
use database_name

// List collections (tables)
show collections

// Dump all documents
db.collection_name.find()

// Pretty print
db.collection_name.find().pretty()

// Search for passwords
db.users.find({}, {username: 1, password: 1})

// Count documents
db.collection_name.countDocuments()

// Server info
db.serverStatus()
db.adminCommand({listDatabases: 1})
```

## nmap scripts

```bash
nmap --script mongodb-info -p 27017 10.10.10.10
nmap --script mongodb-databases -p 27017 10.10.10.10
nmap --script mongodb-brute -p 27017 10.10.10.10
```

## What to look for

| Finding | Impact |
| --- | --- |
| No authentication | Full database access |
| User collections | Credentials, PII |
| Admin database | Server configuration |
| GridFS collections | Stored files |

## See also

- [[36 - Redis Enumeration]] — another NoSQL database
- [[34 - MySQL Enumeration]] — SQL database enumeration
