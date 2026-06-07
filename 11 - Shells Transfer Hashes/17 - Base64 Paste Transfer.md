---
tags: [pentest, transfer, base64, exfiltration, initial-access]
type: cheatsheet
phase: 5
---
# Base64 Paste Transfer

When you have a shell but no network path between machines — encode files as Base64 text and copy-paste through the terminal.

[[00 - README|Folder index]]

## Linux → attacker

```bash
# On target — encode
base64 -w0 /etc/shadow
# Copy the output

# On attacker — decode
echo "<PASTE_BASE64>" | base64 -d > shadow.txt
```

## Attacker → Linux target

```bash
# On attacker — encode
base64 -w0 linpeas.sh

# On target — decode
echo "<PASTE_BASE64>" | base64 -d > linpeas.sh
chmod +x linpeas.sh
```

## Windows → attacker

```powershell
# On target — encode
[Convert]::ToBase64String([IO.File]::ReadAllBytes("C:\path\file.exe"))
# Copy output

# On attacker — decode
echo "<PASTE_BASE64>" | base64 -d > file.exe
```

## Attacker → Windows target

```bash
# On attacker — encode
base64 -w0 nc.exe | xclip -selection clipboard
```

```powershell
# On target — decode
$b64 = "<PASTE_BASE64>"
[IO.File]::WriteAllBytes("C:\Temp\nc.exe", [Convert]::FromBase64String($b64))
```

## Limitations

- Shell buffer limits: very large files may get truncated
- Line wrapping: use `-w0` on Linux to prevent line breaks in output
- For files > 1 MB, use a proper transfer method

> [!tip] Works through anything
> Base64 paste works through jump hosts, restricted terminals, VPNs — anywhere you can paste text. It's slow but universal.

## See also

- [[12 - Python HTTP Server Transfer]]
- [[18 - scp and rsync]]
