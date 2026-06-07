---
tags: [pentest, htb, reference, both]
type: reference
---
# Flag Locations Reference

Where the flags actually live, plus common red herrings.

[[00 - README|Folder index]]

## HackTheBox

### Linux

```bash
# User flag
/home/<user>/user.txt

# Root flag
/root/root.txt

# Helpful one-liners (if you don't know the username yet)
find / -name "user.txt" 2>/dev/null
find / -name "root.txt" 2>/dev/null

# If you have www-data shell and need to find candidate users
cat /etc/passwd | grep -v nologin | grep -v false | cut -d: -f1
ls /home/
```

### Windows

```cmd
:: User flag
type C:\Users\<user>\Desktop\user.txt

:: Root / admin flag
type C:\Users\Administrator\Desktop\root.txt

:: PowerShell variant (works through evil-winrm)
gci -Path C:\Users\ -Filter user.txt -Recurse -ErrorAction SilentlyContinue
gci -Path C:\Users\Administrator\ -Filter root.txt -Recurse -ErrorAction SilentlyContinue

:: If you've inherited a weird user
whoami
dir C:\Users\
```

## TryHackMe / OffSec / labs

THM rooms vary a lot. Common conventions:

```bash
# User flag locations seen:
/home/<user>/user.txt
/home/<user>/flag.txt
/home/<user>/flag1.txt
~/.flag

# Root flag locations seen:
/root/root.txt
/root/flag.txt
/root/flag2.txt
/etc/flag.txt
/srv/flag.txt
```

```cmd
:: Windows flags - similar to HTB, but also:
type C:\flag.txt
type C:\Windows\System32\config\flag.txt
type C:\Users\Public\Desktop\flag.txt
```

## OSCP exam

- User proof: `/home/<user>/proof.txt` or `C:\Users\<user>\Desktop\proof.txt` (rules change per cohort - check current exam guide).
- Root proof: `/root/proof.txt` or `C:\Users\Administrator\Desktop\proof.txt`.
- **Local content** (sometimes required): `/etc/passwd`, `C:\Users\<user>\Desktop\local.txt`, etc.
- Always screenshot the contents alongside `whoami` and `hostname` for proof.

## "Hidden flag" patterns (writeup gold but not required)

```bash
# Loot pass - find all the things HTB box authors plant
find / -type f -size -1k 2>/dev/null | xargs -I {} grep -l "HTB{" {} 2>/dev/null
find / -name "*.txt" -path "*/Desktop/*" 2>/dev/null

# In databases
sqlite3 file.db ".tables"
sqlite3 file.db "SELECT * FROM <tab>;"

# In images (steganography)
exiftool * 2>/dev/null | grep -i "user.txt\|flag"
steghide info image.jpg
```

## Things that look like flags but aren't

- A file called `flag.txt` containing `Not the flag, try harder` - common HTB easter egg, look elsewhere.
- Base64 strings that decode to `Nope.` or `Almost!` - red herring.
- README files in `/opt/` or `/home/<user>/Documents/` - sometimes hints, sometimes noise.

> [!tip] First win when you get a shell
> Run `find / \( -name "user.txt" -o -name "root.txt" -o -name "proof.txt" -o -name "flag.txt" \) 2>/dev/null` immediately. If `user.txt` is *world-readable*, you can submit user without `su`-ing - common on poorly-permissioned boxes.
