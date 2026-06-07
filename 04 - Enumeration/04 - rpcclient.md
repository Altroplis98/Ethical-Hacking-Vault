---
tags: [pentest, enumeration, smb, rpc, rpcclient, recon, windows]
tool: rpcclient
phase: 3
---
# rpcclient

Low-level MSRPC client for Windows/Samba. Direct access to SAM, LSA, and SVCCTL interfaces for detailed enumeration.

[[04 - Enumeration/00 - README|Folder index]]

## Connect

```bash
# Null session
rpcclient -U '' -N 10.10.10.10

# With credentials
rpcclient -U 'user%password' 10.10.10.10

# Domain
rpcclient -U 'DOMAIN/user%password' 10.10.10.10
```

## Key commands

### User enumeration

```text
rpcclient $> enumdomusers           # List all domain users
rpcclient $> queryuser 0x1f4        # Query user by RID (0x1f4 = 500 = Administrator)
rpcclient $> lookupnames admin      # Name → SID
rpcclient $> lookupsids S-1-5-21-...-500   # SID → name
```

### Group enumeration

```text
rpcclient $> enumdomgroups          # List domain groups
rpcclient $> querygroupmem 0x200    # Members of group by RID
rpcclient $> querygroup 0x200       # Group info
```

### Password policy

```text
rpcclient $> getdompwinfo           # Password policy
rpcclient $> getusrdompwinfo 0x1f4  # Per-user policy
```

### Share enumeration

```text
rpcclient $> netshareenum           # List shares
rpcclient $> netshareenumall        # All shares including hidden
rpcclient $> netsharegetinfo share_name  # Share details
```

### Other useful commands

```text
rpcclient $> srvinfo                # Server info (OS, version)
rpcclient $> enumprinters           # Enumerate printers
rpcclient $> enumprivs              # List privileges
```

## RID brute-force (one-liner)

```bash
for i in $(seq 500 1100); do
  rpcclient -U '' -N 10.10.10.10 -c "queryuser 0x$(printf '%x' $i)" 2>/dev/null | grep 'User Name'
done
```

## See also

- [[01 - enum4linux and enum4linux-ng]] — wraps rpcclient in automation
- [[05 - NetExec (nxc)]] — higher-level SMB/RPC tool
