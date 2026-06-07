---
tags: [pentest, enumeration, smtp, email, recon]
tool: smtp-user-enum
phase: 3
---
# smtp-user-enum

Enumerate valid email addresses / usernames on an SMTP server using VRFY, EXPN, or RCPT TO commands.

[[04 - Enumeration/00 - README|Folder index]]

## Install / verify

```bash
which smtp-user-enum || sudo apt install smtp-user-enum -y
```

## Usage

```bash
# VRFY method (most common)
smtp-user-enum -M VRFY -U users.txt -t 10.10.10.10

# RCPT method
smtp-user-enum -M RCPT -U users.txt -t 10.10.10.10

# EXPN method
smtp-user-enum -M EXPN -U users.txt -t 10.10.10.10

# Custom port
smtp-user-enum -M VRFY -U users.txt -t 10.10.10.10 -p 587
```

## Methods explained

| Method | Command | Description |
| --- | --- | --- |
| VRFY | `VRFY user` | Ask server if user exists |
| EXPN | `EXPN user` | Expand mailing list (also reveals users) |
| RCPT | `RCPT TO: user` | Check if server accepts mail for user |

## See also

- [[23 - swaks]] — Swiss Army Knife for SMTP testing
