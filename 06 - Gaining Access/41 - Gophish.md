---
tags: [pentest, gophish, phishing, social-engineering, both, initial-access, web]
tool: gophish
phase: 5
---
# Gophish

Open-source phishing framework with full campaign management: templates, landing pages, tracking, click stats.

[[06 - Gaining Access/00 - README|Folder index]]

## Install + run

```bash
# Get binary
wget https://github.com/gophish/gophish/releases/latest/download/gophish-v0.12.1-linux-64bit.zip
unzip gophish-v0.12.1-linux-64bit.zip -d gophish
cd gophish

# Edit config.json - bind admin to 0.0.0.0 if remote, change default admin password!
sudo ./gophish

# Admin UI: https://localhost:3333
# Default user: admin
# Default pass: gophish (older builds); check logs for first-run password
```

## Campaign components

| Component | Purpose |
| --- | --- |
| **Sending Profile** | SMTP server to send through (your own + DKIM/SPF) |
| **Email Template** | Subject + HTML/text body; tracks `{{.URL}}` placeholder |
| **Landing Page** | Web page victim sees after clicking |
| **Users & Groups** | Target list (email + name) |
| **Campaigns** | Combines all the above + schedule |

## Quick workflow

```text
1. Buy a lookalike domain (corp-it.net for corp.com) → SPF/DKIM/DMARC for it
2. Set up SMTP (Postfix or relay through a service)
3. In Gophish:
     - Sending Profile: SMTP details
     - Landing Page: clone (URL or paste HTML) the real login
     - Email Template: write the lure (IT update, password expiry, etc.)
     - Users: import CSV
     - Campaign: launch with URL = your phishing site
4. Watch Dashboard for clicks / submitted creds
```

## DNS / domain prep

```text
- Register a believable domain (avoid generic .xyz - use .com / .net / .org if budget)
- Wait a few days for "domain reputation" (or use aged domain via providers)
- Set DMARC: v=DMARC1; p=none; rua=mailto:dmarc@yourdomain
- Set SPF: v=spf1 mx ip4:<your-smtp-ip> ~all
- Set DKIM via your mail provider
- Test with mail-tester.com - aim for 9/10+
```

## Reducing detection

- Use Google Safe Browsing / VirusTotal Lookup BEFORE sending - if your domain's already flagged, abort.
- Don't reuse the same campaign template across many engagements.
- Use HTML email, but keep it short - long fancy emails trigger spam filters.

## Combining with Evilginx2

For MFA-protected targets, Gophish hosts the initial lure email, but the landing page link goes to your Evilginx2 instance (which proxies the real login and captures session cookies). See [[42 - Evilginx2 AiTM]].

## When you see X, do Y

| Symptom | Action |
| --- | --- |
| 0 clicks after 24 hours | Email landed in spam. Check spam score (mail-tester.com). Tweak subject + sender. |
| Clicks but no creds | Landing page broken (CSP, missing assets). Re-clone. |
| Browser warning "Deceptive site" | Domain flagged. Get a new one. |
| Microsoft / Google blocking domain quickly | Use a real CA cert + age the domain. Or use one-time URLs (URL randomizer plugin). |

> [!warning] Authorization paper trail
> Phishing requires explicit, written, dated authorization. Keep a copy printed with you during campaigns. Stop immediately if HR / legal contacts you.

## See also

- [[42 - Evilginx2 AiTM]]
