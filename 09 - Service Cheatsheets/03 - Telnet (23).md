---
tags: [pentest, cheatsheet, telnet, service, both]
port: 23
phase: reference
---
# Telnet (23)

[[09 - Service Cheatsheets/00 - README|Folder index]]

## Attacker Mindset

Cleartext protocol. Its presence signals a legacy or embedded system — industrial control, old switches, IoT, forgotten Solaris boxes. Everything including credentials is transmitted in plaintext. If you have network position, you will almost certainly capture creds via sniff. **Common attack vectors:** credential sniffing, MitM, brute force, session hijacking.

## Connect

```bash
telnet $IP
telnet $IP 23
```

## Enumerate

```bash
nmap -sV -sC -p 23 $IP
nmap --script telnet-ntlm-info -p 23 $IP
```

## Brute-force

```bash
hydra -l admin -P passwords.txt telnet://$IP
```

## Notes

- Telnet sends everything in **cleartext** — capture with Wireshark
- Often found on legacy devices (routers, switches, IoT)
- Default credentials are extremely common on telnet services
- Check for banner information disclosure
