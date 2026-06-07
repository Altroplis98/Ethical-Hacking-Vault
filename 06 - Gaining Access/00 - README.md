---
tags: [pentest, gaining-access, exploitation, moc, initial-access]
phase: 5
---
# 06 - Gaining Access

Turn weaknesses into a foothold: exploit a vuln, abuse a misconfig, or use valid creds.

[[00 - Vault Index|Home]] · Prev: [[05 - Vulnerability Analysis/00 - README|Vuln Analysis]] · Next: [[07 - Post-Exploitation/Linux/00 - README|Post-Ex]]

Use hashid -j to decode hashes
## Files in this folder

### Online password attacks
- [[01 - THC Hydra]]
- [[02 - Medusa]]
- [[03 - Ncrack]]
- [[04 - NetExec Password Attacks]]
- [[05 - Password Spraying Strategy]]

### Offline password attacks
- [[06 - Hashcat Core]]
- [[07 - John the Ripper Core]]

### Public exploits
- [[08 - Metasploit Framework Workflow]]
- [[09 - msfvenom Payload Cookbook]]
- [[10 - Searchsploit to Working Exploit]]

### Network credential attacks
- [[11 - Responder LLMNR NBT-NS Poisoning]]
- [[12 - ntlmrelayx]]
- [[13 - Impacket Suite Overview]]
- [[14 - secretsdump]]

### Kerberos abuse
- [[15 - Kerberoasting]]
- [[16 - AS-REP Roasting]]
- [[17 - Pass-the-Hash]]
- [[18 - Pass-the-Ticket]]
- [[19 - Overpass-the-Hash]]

### AD-specific
- [[20 - DCSync]]
- [[21 - Certipy ESC Attacks]]
- [[22 - Coercion (PetitPotam Coercer)]]
- [[23 - ZeroLogon Reference]]
- [[24 - PrintNightmare Reference]]
- [[25 - NoPac sAMAccountName]]

### Remote shells via creds
- [[26 - evil-winrm]]
- [[27 - psexec wmiexec smbexec]]
- [[28 - xfreerdp]]

### Web exploitation
- [[29 - SQLMap]]
- [[30 - XSStrike]]
- [[31 - Commix]]
- [[32 - Manual SQLi Payloads]]
- [[33 - XSS Payloads]]
- [[34 - SSRF Cheats]]
- [[35 - XXE Payloads]]
- [[36 - Deserialization (ysoserial)]]
- [[37 - File Upload Bypass]]
- [[38 - LFI to RCE Patterns]]
- [[39 - Tomcat Manager Exploit Chain]]

### Social engineering
- [[40 - SET Toolkit]]
- [[41 - Gophish]]
- [[42 - Evilginx2 AiTM]]

## Decision tree

```text
Have valid creds?
   YES → try every service (SMB/WinRM/RDP/SSH/MSSQL) with NetExec spray
   NO  → ↓

Have hash?
   YES → crack (hashcat) OR pass-the-hash directly (psexec/winrm/smb)
   NO  → ↓

Auth bypass available?
   YES → use it
   NO  → ↓

Public exploit for service version?
   YES → vet PoC, test in lab, deploy
   NO  → ↓

Weak creds attempt? (spray, not brute)
   YES → spray with caution; check lockout policy first
   NO  → ↓

Web app on port 80/443?
   YES → manual testing + sqlmap + nuclei
   NO  → revisit enumeration; you missed something
```

> [!warning] Lockout discipline
> Spraying (1 password against many users) >> brute-forcing (many passwords against 1 user). Run `nxc smb <ip> -u user -p '' --pass-pol` to read the lockout threshold before you spray.
