---
tags: [pentest, cheatsheet, memcached, service, both]
port: 11211
phase: reference
---
# Memcached (11211)

[[09 - Service Cheatsheets/00 - README|Folder index]]

## Connect

```bash
telnet $IP 11211
nc $IP 11211
```

## Commands

```text
stats                    # server stats
stats items              # item statistics
stats cachedump 1 100    # dump keys from slab 1
get key_name             # retrieve value
version                  # version info
```

## Enumerate

```bash
nmap -sV -p 11211 $IP
nmap --script memcached-info -p 11211 $IP

# Dump all keys (memcdump)
memcdump --servers=$IP
memccat --servers=$IP key_name
```

## Common findings

- No authentication (default) → dump all cached data
- Cached session tokens, credentials, API keys
- DDoS amplification vector (UDP reflection)
