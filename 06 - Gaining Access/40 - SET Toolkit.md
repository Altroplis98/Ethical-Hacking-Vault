---
tags: [pentest, set, social-engineering, phishing, both, initial-access]
tool: setoolkit
phase: 5
---
# Social-Engineer Toolkit (SET)

Menu-driven phishing / spear-phishing / website cloning framework.

[[06 - Gaining Access/00 - README|Folder index]]

## Launch

```bash
sudo setoolkit
```

## Main menu navigation

```text
1) Social-Engineering Attacks
   1) Spear-Phishing Attack Vectors
   2) Website Attack Vectors
      1) Java Applet Attack
      2) Metasploit Browser Exploit
      3) Credential Harvester  ← classic phishing page clone
      4) Tabnabbing
      5) Web Jacking
      6) Multi-Attack
   3) Infectious Media Generator   ← USB drop
   4) Create a Payload + Listener
   5) Mass Mailer
   6) Arduino-Based Attack Vector
   7) Wireless AP Attack Vector
   8) QRCode Generator
   9) Powershell Attack Vectors
```

## Most common workflow - credential harvester (clone login page)

```text
setoolkit > 1 > 2 > 3 > 2 (Site Cloner)
IP for POST back: 10.10.14.5
URL to clone: https://target-org.okta.com/
```

SET hosts a copy of that login page at `http://10.10.14.5/`. When victim enters creds, they're captured to `/var/www/html/harvester_<date>.txt`.

## Important notes for real engagements

- SET sets up a basic HTTP server. For better phishing, run Apache/Nginx with a real TLS cert (Let's Encrypt).
- Use `gophish` or `evilginx2` for real campaigns - SET is dated.
- Get explicit written approval before any phishing.

## When to use SET vs alternatives

| Need | Tool |
| --- | --- |
| Quick credential harvester for one-off test | SET |
| Single-page social media login clone (Instagram, FB, etc.) | **ShellPhish** (see below) |
| Multi-target campaign with tracking + templates | [[41 - Gophish]] |
| MFA bypass (session cookie capture) | [[42 - Evilginx2 AiTM]] |
| USB / HID attacks | SET has Arduino module |
| Wireless rogue AP | wifiphisher / airgeddon |

## ShellPhish — lightweight alternative for single-page clones

ShellPhish ships ~